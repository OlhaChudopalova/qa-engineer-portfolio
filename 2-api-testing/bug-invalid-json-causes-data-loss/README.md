# BUG: API accepts invalid JSON with duplicated keys leading to data loss

---

## Endpoints
- POST `/api/v1/executor/post/profile-data/`  
- PUT `/api/v1/executor/update/profile-data/`  
- GET `/api/v1/executor/get/profile-data/`

---

## Description

API приймає невалідну структуру JSON у блоці `contacts_data`, що призводить до втрати контактних даних та непередбачуваної поведінки системи.

---

## Preconditions
Користувач авторизований та має можливість заповнювати профіль через API.

---

## Steps to Reproduce
1. Надіслати POST-запит на `/api/v1/executor/post/profile-data/` з дубльованими ключами в одному об’єкті масиву `contacts` або `networks`.  
2. Виконати GET-запит на `/api/v1/executor/get/profile-data/`.  
3. Перевірити, які контакти збережені та відображаються на фронтенді.

---

## Expected Result
API має відхиляти невалідну структуру JSON та повертати помилку 4xx з повідомленням про некоректний формат даних.

---

## Actual Result
API повертає 200 OK та зберігає лише останнє значення з дубльованих ключів. В результаті відбувається втрата даних, а збережені контакти залежать від порядку ключів у тілі запиту.

---

## Impact / Risks
- Втрата контактних даних користувача.  
- Некоректне відображення профілю на фронтенді.  
- Непередбачувана поведінка API залежно від порядку ключів.  
- Порушення контракту API та стандарту JSON.

---

## Recommendation
Додати серверну валідацію для:  
- `contacts_data.contacts`  
- `contacts_data.networks`  

та повертати помилку 4xx у разі дубльованих ключів або неправильної структури об’єкта.

---

## Additional Details

### Swagger-defined structure
Блок `contacts_data` містить масиви об’єктів, де кожен об’єкт – окремий контакт або соцмережа:

```json
"contacts_data": {
  "contacts": [
    { "contact_type": "phone", "contact_value": "639797753" },
    { "contact_type": "telegram", "contact_value": "639797753" },
    { "contact_type": "whatsapp", "contact_value": "639797753" }
  ],
  "networks": [
    { "network": "LinkedIn", "link": "https://www.linkedin.com/in/vitalina-sitalo-13b514136/" },
    { "network": "GitHub", "link": "https://github.com/alice-dev" }
  ]
}

```

Current API behavior
1. Для contacts
Приклад A – втрачається whatsapp та telegram:

```json

"contacts": [
  { "contact_type": "whatsapp",
    "contact_value": "639797753",

    "contact_type": "telegram",
    "contact_value": "639797753",

    "contact_type": "phone",
    "contact_value": "639797753"
  }
]

```
Результат GET:

```json

{ "contact_type": "phone",
  "contact_value": "639797753" }

```
Приклад B (зміна порядку – інший результат):

```json

"contacts": [     
  { "contact_type": "telegram",
    "contact_value": "639797753",

    "contact_type": "phone",
    "contact_value": "639797753",

    "contact_type": "whatsapp",
    "contact_value": "639797753"
  }
]

```
Результат GET:

```json

{ "contact_type": "whatsapp",
  "contact_value": "639797753" }

```
2. Для networks
Приклад A (LinkedIn втрачається):

```json

"networks": [ 
  { "network": "LinkedIn",
    "link": "https://linkedin.com/linked",
    "network": "GitHub",
    "link": "https://github.com/gh" }
]

```
Результат GET:

```json

{ "network": "GitHub", 
  "link": "https://github.com/gh" }

```
Приклад B (інший порядок – інший результат):

```json

"networks": [
  { "network": "GitHub",
    "link": "https://github.com/gh",

    "network": "LinkedIn",
    "link": "https://linkedin.com/linked" }
]

```
Результат GET:

```json

{ "network": "LinkedIn", 
  "link": "https://linkedin.com/linked" }

```
Key Observation:
Key Observation Черговість ключів прямо визначає, які контакти потраплять у БД і що відобразиться у фронтенді. Залежно від порядку ключів у тілі запиту в БД та на фронті зберігатиметься різний контакт. 

Сервер приймає навіть більш “екстремальні” випадки дублювання ключів у одному об’єкті:

```json

{ "contact_type": "telegram",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "phone",
  "contact_value": "639797753",
  "contact_type": "whatsapp",
  "contact_value": "639767753",
  "contact_type": "whatsapp",
  "contact_value": "639767753",
  "contact_type": "whatsapp",
  "contact_value": "639767753" }

```
GET-запит повертає лише останнє значення:

```json

{ "contact_type": "whatsapp",
  "contact_value": "639767753" }

```

### Attachments in screenshots folder

**1. Invalid POST request with duplicated keys**
**2. GET response showing data loss** 
**3. Frontend displays only last saved contact**


#### Last_github – Lost LinkedIn: 
- `post_last_github.jpg`
- `get_last_github.jpg` 
- `front_last_github.jpg` 


#### Last_phone – Lost other contacts: 
- `post_last_phone.jpg` 
- `get_last_phone.jpg` 
- `front_last_phone.jpg` 


#### Last_whatsapp – Lost other contacts: 
- `post_last_whatsapp.jpg` 
- `get_last_whatsapp.jpg` 
- `front_last_whatsapp.jpg`


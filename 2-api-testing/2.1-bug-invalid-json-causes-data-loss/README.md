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
**LinkedIn-профіль** є обов’язковим полем для збереження профілю.

---

## Steps to Reproduce
1. Надіслати POST-запит на `/api/v1/executor/post/profile-data/` з дубльованими ключами в одному об’єкті масиву `contacts` або `networks`.  
2. Виконати GET-запит на `/api/v1/executor/get/profile-data/`.  
3. Перевірити, які контакти збережені та відображаються на фронтенді.

---

## Expected Result
API має:
- відхиляти невалідну структуру JSON;
- повертати помилку 4xx у разі дубльованих ключів або неправильної структури;
- **повертати помилку 4xx, якщо відсутнє обов’язкове поле LinkedIn**;
- не дозволяти зберігати профіль без посилання на LinkedIn.

---

## Actual Result
API повертає 200 OK та зберігає лише останнє значення з дубльованих ключів.  
В результаті:
- відбувається втрата даних;
- збережені контакти залежать від порядку ключів;
- **профіль може бути збережений без обов’язкового LinkedIn**, що порушує бізнес-логіку.

---

## Impact / Risks
- Можливість обходу обов’язкової бізнес-вимоги (LinkedIn) через маніпуляцію структурою JSON.
- Збереження некоректних або неповних профілів у системі.
- Некоректне відображення профілю на фронтенді.  
- Непередбачувана поведінка API залежно від порядку ключів.  
- Порушення контракту API та стандарту JSON.

---

## Recommendation
Додати серверну валідацію для:  
- `contacts_data.contacts`  
- `contacts_data.networks`  

та повертати помилку 4xx у разі:
- дубльованих ключів;
- неправильної структури;
- **відсутності обов’язкового поля LinkedIn.**

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

### Current API behavior
#### 1. Для "contacts"
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
#### 2. Для "networks"
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
### Key Observation:  
Черговість ключів прямо визначає, які контакти потраплять у БД і що відобразиться у фронтенді. Залежно від порядку ключів у тілі запиту в БД та на фронті зберігатиметься різний контакт.  
Сервер приймає навіть більш “екстремальні” випадки дублювання ключів в одному об’єкті:

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

### 📎 Attachments (screenshots folder)

🔸  Invalid POST request with duplicated keys  
🔸  GET response showing data loss  
🔸  Frontend displays only last saved contact


🔹 **last_github_lost_linkedin:** [01_post_last_github.jpg](screenshots/last_github_lost_linkedin/01_post_last_github.jpg), [02_get_last_github.jpg](screenshots/last_github_lost_linkedin/02_get_last_github.jpg), [03_front_last_github.jpg](screenshots/last_github_lost_linkedin/03_front_last_github.jpg)

🔹 **last_phone:** [01_post_last_phone.jpg](screenshots/last_phone/01_post_last_phone.jpg), [02_get_last_phone.jpg](screenshots/last_phone/02_get_last_phone.jpg), [03_front_last_phone.jpg](screenshots/last_phone/03_front_last_phone.jpg)

🔹 **last_whatsapp:** [01_post_last_whatsapp.jpg](screenshots/last_whatsapp/01_post_last_whatsapp.jpg), [02_get_last_whatsapp.jpg](screenshots/last_whatsapp/02_get_last_whatsapp.jpg), [03_front_last_whatsapp.jpg](screenshots/last_whatsapp/03_front_last_whatsapp.jpg)


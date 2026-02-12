#  Original Bug Report

**Title:** The POST request updates profiles in any status and sets them to PENDING  
**Priority:** High    
**Severity:** High  

---

##  Description
Endpoint `POST /api/v1/executor/post/profile-data/` не лише створює, а й оновлює дані профілю, фактично працюючи як PUT/UPDATE. Окрім цього, він встановлює статус `Pending` у всіх випадках після виконання запиту.  
Це порушує логіку модерації та принципи REST.


---

##  Environment
- **API:** http://api.stafferia.com  
- **Postman:** v11.68.4-251023-0629  
- **Browser:** Google Chrome 141.0.7390.122  
- **OS:** Windows 11 Home 25H2  

---

##  Endpoint
`POST /api/v1/executor/post/profile-data/`

---

##  Preconditions
Профіль існує у будь-якому статусі:  
DRAFT, PENDING, PART_BLOCK, TIMER_BLOCK, FULL_ BLOCK, APPROVED, CONFIRMED.

---

##  Steps to Reproduce
1. Відправити `POST /api/v1/executor/post/profile-data/` з валідним JSON для статусу `DRAFT`.
2. Повторити для всіх статусів профілю.
3. Перевірити статус профілю після виконання POST-запиту.

---

##  Expected Result
Endpoint `POST /api/v1/executor/post/profile-data/`:
- дозволений для статусів `DRAFT` / `PART_BLOCK` з переходом у статус `Pending`;
- у всіх інших статусах має повертати 4xx помилку.

---

## Actual Result
Endpoint `POST /api/v1/executor/post/profile-data/` створює та оновлює профіль у будь-якому статусі та переводить його в `PENDING`.

---

##  Impact
- Порушення логіки модерації.
- Некоректний workflow.
- POST працює як PUT.

---

## Suggested Fix
Endpoint `POST /api/v1/executor/post/profile-data/` застосовувати виключно для статусів `DRAFT` / `PART_BLOCK` та присвоювати статус `PENDING`.  
В інших статусах повертати 4xx помилку.

---

##  Attachments
Скріншоти демонструють проблемну поведінку та контекст багу:
- **Bug screenshots** (до виправлення, для баг-репорту): [./screenshots/bug](./screenshots/bug)  
Для наглядності:
- У полі `first_name` під час POST/PUT запиту введено поточний статус профілю (наприклад, `Mikle_POST_Confirmed`), який відображається на фронтенді та в Admin Panel.  
- Кнопка **Pending Approval** підтверджує перехід статусу в `Pending` на фронтенді, а в Admin Panel явно відображено статус `Pending`.

# Proposed Improvement


## Rationale

Тестування показало, що PUT-запит фактично дублює функцію POST і дозволяє редагувати профіль у будь-якому статусі без обмежень, що суперечить бізнес-логіці та створює потенційні ризики для цілісності даних.  

Це створює кілька проблем:
- Відсутнє чітке розмежування між створенням (POST) і редагуванням (PUT)
- Ризик неконтрольованих змін, що може призвести до «сміття» в базі даних
- Порушується workflow модерації, оскільки профілі можуть змінюватися без контролю

---

## Observed Behavior of POST/PUT Before Improvement

| Status      | POST (current behavior) | PUT (current behavior) | Notes | Status After |
|-------------|---------------------------|--------------------------|----------|--------------|
| DRAFT       | ✅ Yes                    | ✅ Yes                   | PUT не має оновлювати нестворений ресурс (BUG) | PENDING (POST) |
| PENDING     | ✅ Yes                    | ✅ Yes                   | Обидва запити дозволяють редагування під час модерації (BUG) | No change |
| PART_BLOCK  | ✅ Yes                    | ✅ Yes                   | Відкрито питання: згідно бізнес-логіки – POST; згідно API — PUT | No change |
| FULL_BLOCK  | ✅ Yes                    | ✅ Yes                   | Неможливе редагування заблокованого профілю (BUG) | No change |
| APPROVED    | ✅ Yes                    | ✅ Yes                   | POST не має оновлювати вже затверджений профіль (BUG) | No change |
| TIMER_BLOCK | ✅ Yes                    | ✅ Yes                   | POST не має оновлювати вже затверджений профіль (BUG) | No change |
| CONFIRMED   | ✅ Yes                    | ✅ Yes                   | POST не має оновлювати вже затверджений профіль (BUG) | No change |
| DELETED     | ❌ No                     | ❌ No                    | Профіль видалено                                      | No change |
---
### PUT Updates Profile in Any Status

**Examples:**

![PUT Updates Profile in Any Status](./screenshots/improvement/put_draft_old.jpg)
![PUT Updates Profile in Any Status](./screenshots/improvement/put_pending_old.jpg)

> Скриншоти демонструють, що PUT оновлює профіль у будь-якому статусі та не накладає обмежень відповідно до бізнес-логіки.

Ця неконтрольована поведінка показала потребу у впровадженні системного Improvement.   
Нові правила CRUD були запропоновані для того, щоб забезпечити контрольоване створення та редагування профілів відповідно до їх статусу.   
Виходячи з цього, були визначені основні цілі покращення:



---



##  Goal
- Уникнути хаотичного оновлення даних.
- Чітко розділити створення профілю (POST) та редагування (PUT).
- Уточнити використання POST для статусів `DRAFT` / `PART_BLOCK`.

---

##  Endpoints
- `POST /api/v1/executor/post/profile-data/`  
- `PUT /api/v1/executor/update/profile-data/`

---

## Business Rules Proposal

### POST (Profile Creation)
**Allowed statuses:**
- `DRAFT`  
- `PART_BLOCK` (дискусійно)  

> У цих випадках POST означає створення та подання профілю на модерацію, після чого статус змінюється на `PENDING`.

**Forbidden statuses:**  
`PENDING`, `TIMER_BLOCK`, `FULL_BLOCK`, `APPROVED`, `CONFIRMED`, `DELETED`

---

### PUT (Profile Update)
**Allowed statuses:**
- `PART_BLOCK` (дискусійно)  
- `APPROVED`  
- `TIMER_BLOCK`  
- `CONFIRMED`  

> У цих статусах користувач уже пройшов модерацію, тому редагування профілю є коректним сценарієм.

**Forbidden statuses:**  
`DRAFT`, `PENDING`, `FULL_BLOCK`, `DELETED`


---

##  Team Discussion: POST vs PUT for PART_BLOCK

### Бізнес-логіка
- `PART_BLOCK` – повторна чернетка після фідбеку модерації, не фінальний ресурс.  
- Є два типи чернеток:  
  - `DRAFT` – первинна чернетка  
  - `PART_BLOCK` – повторна чернетка після зауважень модерації  

> Обидві чернетки: не є затвердженим профілем, не існують як фінальний ресурс, перебувають у процесі створення.  
> Тому POST для `PART_BLOCK` логічно сприймається як продовження створення, а не редагування.

### API Perspective
- Якщо профіль уже створений через POST і повернуто `201 Created`, ресурс вважається існуючим.  
- Усі подальші правки після фідбеку модерації виконуються через PUT.


## CRUD Rules Table (Proposed)

| Status      | POST | PUT | Note                                                                 | Status After |
|------------|------|-----|----------------------------------------------------------------------|--------------|
| DRAFT      | ✅ Yes | ❌ No | Створення нового ресурсу; редагування через PUT неможливе, бо профіль ще не існує в БД | PENDING |
| PENDING    | ❌ No | ❌ No | Профіль на модерації; зміни заборонено | No change |
| PART_BLOCK |  Discussed |  Discussed | API: POST → PUT, Бізнес-логіка: продовження створення після фідбеку модерації | PENDING |
| FULL_BLOCK | ❌ No | ❌ No | Профіль заблоковано | No change |
| APPROVED   | ❌ No | ✅ Yes | Редагування дозволено | No change |
| TIMER_BLOCK| ❌ No | ✅ Yes | Редагування дозволено | No change |
| CONFIRMED  | ❌ No | ✅ Yes | Редагування дозволено | No change |
| DELETED    | ❌ No | ❌ No | Профіль видалено | No change |

---

## Final Decision
Після дискусії між BE, FE, BA та QA була впроваджена нова логіка:
- чітке розділення POST (створення) і PUT (редагування);
- жорсткі правила доступу залежно від статусу.

---

## Testing Results (After Implementation)

| Status      | POST HTTP | POST Result | PUT HTTP | PUT Result | Note | Status After |
|------------|-----------|------------|----------|-----------|------|--------------|
| DRAFT      | 201       | ✅ Created, відправлено на модерацію | 422      | ❌ Forbidden | Створення профілю | PENDING |
| PENDING    | 422       | ❌ Forbidden | 422      | ❌ Forbidden | Профіль на модерації; зміни заборонено | No change |
| PART_BLOCK | 422       | ❌ Forbidden | 200      | ✅ Allowed | Редагування дозволено 1 раз | PENDING |
| TIMER_BLOCK| 422       | ❌ Forbidden | 200      | ✅ Allowed | Редагування дозволено | No change |
| FULL_BLOCK | 422       | ❌ Forbidden | 422      | ❌ Forbidden | Профіль заблоковано | No change |
| APPROVED   | 422       | ❌ Forbidden | 200      | ✅ Allowed | Редагування дозволено | No change |
| CONFIRMED  | 422       | ❌ Forbidden | 200      | ✅ Allowed | Редагування дозволено | No change |
| DELETED    | 422       | ❌ Forbidden | 422      | ❌ Forbidden | Профіль видалено | No change |

---

##  Final Outcomes / Achievements
- Унеможливлено хаотичне оновлення профілів та чітко розділено створення (POST) і редагування (PUT)  
- Усі оновлення перевіряють статус, правила доступу та повертають коректний статус згідно з бізнес-логікою  
- Запобігає логічним помилкам та підвищує надійність системи  
- Досягнуто завдяки аналізу бізнес-логіки, тестуванню API відповідно до REST-принципів та ефективній комунікації з командами (BE/FE/BA/QA)

---

## Attachments

Скріншоти демонструють порівняння поведінки API до та після впровадження нових правил CRUD:


### PUT – оновлення профілю
- Before: [put_draft_old.jpg](./screenshots/bug/put_draft_old.jpg), [put_pending_old.jpg](./screenshots/bug/put_pending_old.jpg)
- After: [put_draft_new.jpg](./screenshots/improvement/put_draft_new.jpg), [put_pending_new.jpg](./screenshots/improvement/put_pending_new.jpg)

### POST — створення профілю
- Before: [post_confirmed_old.jpg](./screenshots/bug/post_confirmed_old.jpg)
- After: [post_confirmed_new.jpg](./screenshots/improvement/post_confirmed_new.jpg)


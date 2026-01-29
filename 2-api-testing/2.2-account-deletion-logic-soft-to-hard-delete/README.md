# Case: Account Deletion Logic – From Soft Delete to Hard Delete

### 🔸 Value for Business & QA
Демонстрація повного QA циклу:  
**Detection → Analysis → Recommendation → Validation**

---

### 🔸 Background
Під час тестування endpoint видалення акаунту було виявлено, що система використовує soft delete, через що:
- повторна реєстрація з тим самим email зберігає user_id;
- частково відновлюються персональні дані;
- дані можуть переноситись між різними ролями (Company → Executor).

Це створювало:
- ризики для безпеки та приватності;
- неочікувану поведінку з точки зору UX;
- складність при масштабуванні multi-role логіки.

---

## 🔹 Initial Testing: Soft Delete Behavior

## 1️⃣ Scenario: Delete Company Profile – Same role re-registration (Company → Company)

**Environment**  
- Tool: Postman  
- Browser: Google Chrome 141.0.7390.122 (64-bit)  
- OS: Windows 11 Home 25H2  
- URL: stafferia.com  

**Endpoints**  
- DELETE: `/api/v1/user/delete/`  
- GET: `/api/v1/customer/get/profile-data/`  

**Preconditions**  
- Компанія зареєстрована та активована  
- Company Profile заповнений валідними даними  

**Test Steps**
1. Отримати дані профілю компанії (GET /customer/get/profile-data)  
2. Видалити акаунт (DELETE /user/delete)  
3. Переконатися, що login недоступний  
4. Зареєструвати нового користувача з тим самим email  
5. Активувати акаунт  
6. Увійти в систему  
7. Отримати дані профілю після повторної реєстрації  

**Expected Result**
- Option A – Hard delete: новий профіль, без відновлення даних  
- Option B – Soft delete: поведінка чітко задокументована, користувач поінформований про можливість відновлення даних  

**Actual Result**
- Повторна реєстрація дозволена  
- Збережено `first_name`, `last_name`  
- Інші поля очищені (`null`)  
- Часткове відновлення профілю  

### Token Decoding Analysis
- декодування token: user_id до та після видалення однаковий  
- Новий user не створюється – відновлюється попередній  

### Issue / Observation
Після видалення акаунту та повторної реєстрації з тим самим email система:
- не створює нового користувача;
- зберігає той самий user_id;
- частково відновлює персональні дані.

Це:
- не відповідає очікуванням користувача;
- створює ризики для data privacy;
- ускладнює multi-role логіку;
- не має прозорої комунікації в UX.

### Recommendation

1. **Перевірка бізнес-логіки видалення акаунту**
   - Чи має повторна реєстрація створювати нового користувача (user_id)?
   - Які поля повинні бути очищені при видаленні акаунту?

2. **Вибір стратегії видалення та повідомлення користувача**
   - **Hard delete:** повне видалення акаунту та створення нового user при повторній реєстрації.
   - **Soft delete / анонімізація:** збереження user_id та частини даних, з явним повідомленням користувача про можливе відновлення акаунту та які дані залишаються.

---

## 2️⃣ Scenario: Soft Delete – Cross-role re-registration (Company → Executor)

Після soft delete Company-профілю користувач повторно реєструється з тим самим email, але з роллю Executor.

### Result
- Executor-профіль створюється з тим самим user_id  
- Частина Company-даних автоматично переноситься  

### Token Decoding Analysis
- user_id у токенах при зміні ролі не змінюється  

### Potential Impacts

**Technical**
- Ризики при реалізації multi-role логіки  
- Можливі проблеми з правами доступу та ізоляцією даних  

**Business / UX**
- Неочікуване перенесення даних між ролями  
- Часткова втрата даних без попередження  
- Відсутність прозорої комунікації  

### Conclusion
- user_id не прив’язаний до ролі, прив’язаний до email  
- Дані можуть переноситись між ролями без явної згоди користувача  

### Business Logic Questions to Clarify

Якщо це очікувана поведінка:
- Чи означає це, що один email = один user_id назавжди?
- Які поля дозволено переносити між ролями?
- Які поля мають бути очищені?

Якщо ні:
- Чи має нова реєстрація створювати нового user?
- Чи потрібен hard delete або анонімізація?

---

## 🔹 Decision: Change from Soft Delete to Hard Delete
Після обговорення та донесення інформації до BA та BE команди було прийнято рішення:  
- **Замінити soft delete на hard delete.**


---

## 🔹 Retesting After Logic Change (Hard Delete)

### Test Table: Results of Full Account Deletion Scenario (Customer)

| №  | Endpoint                                   | Method | Опис                                     | Очікувано                         | Фактично                         | Статус |
|----|--------------------------------------------|--------|------------------------------------------|-----------------------------------|-----------------------------------|--------|
| 1.1| /api/v1/customer/get/profile-data/         | GET    | Отримати customer-профіль до видалення    | 200 OK, профіль доступний         | 200 OK, дані повернуто            | ✅ Pass |
| 1.2| /api/v1/user/                              | DELETE | Hard delete акаунту                       | 204 No Content                    | 204 No Content                    | ✅ Pass |
| 1.3| /api/v1/user/ (повторно)                   | DELETE | Повторне видалення                        | 401 Unauthorized / User not found | 401 Unauthorized / User not found | ✅ Pass |
| 1.4| /api/v1/customer/get/profile-data/         | GET    | Доступ до профілю після видалення         | 401 Unauthorized                  | 401 Unauthorized                  | ✅ Pass |
| 1.5| /api/v1/user/login                         | POST   | Логін customer (акаунт видалений)         | 404 Not Found                     | 404 Not Found                     | ✅ Pass |
| 1.6| /api/v1/user/register                      | POST   | Реєстрація з тим самим email (New role)   | 200 OK, verification email sent   | 200 OK, verification email sent   | ✅ Pass |
| 1.7| /api/v1/user/activate                      | POST   | Активація нового акаунту                  | 200 OK                            | 200 OK                            | ✅ Pass |
| 1.8| /api/v1/user/login                         | POST   | Логін нового executor-акаунту             | 200 OK, новий UUID                | 200 OK, новий UUID                | ✅ Pass |
| 1.9| /api/v1/customer/get/profile-data/         | GET    | Доступ до customer-профілю з executor токеном | 404 Not Found                 | 404 Not Found                     | ✅ Pass |
| 2.0| /api/v1/user/info/                         | GET    | Отримати executor-профіль                 | 200 OK, новий UUID                | 200 OK, новий UUID                | ✅ Pass |
| 2.1| /api/v1/executor/get/profile-data/         | GET    | Отримати executor профіль (основний)      | 200 OK, немає зайвої інфо         | 200 OK, немає зайвої інфо         | ✅ Pass |
| 2.2| /api/v1/executor/executor-profiles/{uuid}  | GET    | Отримати executor за новим UUID           | 200 OK                            | 200 OK                            | ✅ Pass |

---

## 🔹 Token Decoding Analysis After Fix
- новий акаунт отримує новий UUID  
- дані та ідентичність не переносяться  
- ролі більше не “успадковують” ідентичність попереднього user  

---

### 🔹 Final Conclusion
Система перейшла від:
- часткового відновлення користувача;
- переносу даних між ролями;
- збереження ідентичності після видалення;

до:
- повного видалення акаунту;
- створення нового user при повторній реєстрації;
- прозорої логіки ролей та усунення ризиків крос-ролевого витоку даних;
- відповідності принципам data privacy;
- покращення масштабованості ролей.

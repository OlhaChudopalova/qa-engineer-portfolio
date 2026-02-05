# API End-to-End Flow – Test Summary Table

| # | Endpoint | Method | Purpose | Expected Result | Actual Result | Status |
|---|---------|--------|--------|----------------|---------------|--------|
| 1 | /api/v1/user/login/ | POST | Login as executor | 200 OK, access token, role=executor | 200 OK, token + role=executor | ✅ Pass |
| 2 | /api/v1/executor/post/profile-data/ | POST | Create executor profile | 201 | 201 Created, status=pending | ✅ Pass |
| 3 | /api/v1/user/ | DELETE | Delete user account | 204 No Content | 204 No Content | ✅ Pass |
| 4 | /api/v1/user/login/ | POST | Login after deletion | 404 Not Found | 404 Not Found | ✅ Pass |
| 5 | /api/v1/user/ | DELETE | Repeat delete | 401 | 401 | ✅ Pass |
| 6 | /api/v1/executor/executor-profiles/{UUID} | GET | Get deleted executor profile | 404 Not Found | 404 Not Found | ✅ Pass |
| 7 | /api/v1/executor/executor-profiles/search | GET | Search deleted profile | Not found in catalog | 200 OK, profile absent | ✅ Pass |
| 8 | /api/v1/user/register | POST | Re-register executor (same email) | 200 OK, verification sent | 200 OK, email sent | ✅ Pass |
| 9 | /api/v1/user/activate | POST | Activate account | 200 OK | 200 OK | ✅ Pass |
| 10 | /api/v1/user/login/ | POST | Login after re-registration | 200 OK, role=executor | 200 OK, role=executor | ✅ Pass |
| 11 | /api/v1/executor/executor-profiles/{OLD_UUID} | GET | Access old profile | 404 Not Found | 404 executor_profile_not_found | ✅ Pass |
| 12 | /api/v1/user/info | GET | Get user info | 200 OK, New UUID, status=draft | 200 OK, New UUID, status=draft | ✅ Pass |
| 13 | /api/v1/executor/get/profile-data | GET | Get new executor profile | 200 OK, Empty profile | 200 OK, Empty profile | ✅ Pass |
| 14 | /api/v1/executor/executor-profiles/{NEW_UUID} | GET | Get profile via other endpoint | 200 OK, Empty profile | 200 OK, Empty profile | ✅ Pass |
| 15 | /api/v1/executor/post/profile-data/ | POST | Re-create executor profile | 201 Created | 201 Created | ✅ Pass |
| 16 | /api/v1/executor/get/profile-data | GET | Get filled executor profile | 200 OK, All fields filled | 200 OK, All fields filled | ✅ Pass |
| 17 | /api/v1/user/ | DELETE | Delete account again | 204 No Content | 204 No Content | ✅ Pass |
| 18 | /api/v1/user/ | DELETE | Repeat delete | 401 Unauthorized | 401 Unauthorized | ✅ Pass |
| 19 | /api/v1/user/register | POST | Register as customer (same email) | 200 OK, email sent | 200 OK, email sent | ✅ Pass |
| 20 | /api/v1/user/activate | POST | Activate customer account | 200 OK | 200 OK | ✅ Pass |
| 21 | /api/v1/user/login/ | POST | Login as customer | 200 OK, role=customer | 200 OK, role=customer | ✅ Pass |
| 22 | /api/v1/customer/get/profile-data | GET | Get customer profile | Clean customer profile, no executor data | Clean customer profile | ✅ Pass |

---

# 1. Functional Scenarios

## 1.1 Profile creation, retrieval, and update
- Створення профілю (№2, №15)  
- Отримання порожнього профілю після реєстрації (№13, №14)  
- Отримання заповненого профілю (№16)  
- Отримання інформації користувача після повторної реєстрації (№12, перевірка нового UUID та статусу draft)  

## 1.2 Authentication & Login
- Логін як executor (№1)  
- Логін після видалення акаунту (№4)  
- Логін після повторної реєстрації (№10)  
- Логін як customer (№21)  

## 1.3 CRUD Operations
- Видалення акаунту (№3, №17)  
- Повторне видалення (№5, №18)  
- Повторна реєстрація executor після видалення (№8)  
- Активація акаунту (№9)  
- Реєстрація та активація customer (№19, №20)  

## 1.4 Role-based profile isolation
- Старий профіль executor недоступний (№11)  
- Новий профіль executor створений і доступний (№14, №16)  
- Профіль customer не містить даних executor (№22)  

---

# 2. Edge & Security Scenarios
- Повторна реєстрація з тим самим email після видалення (№8–№10) — перевірка нового UUID та статусу draft  
- Перемикання ролей після видалення (executor → customer) (№19–№22) — перевірка ізоляції даних та ролей  
- Кілька спроб видалення акаунту (№5, №18) — перевірка коректних помилок та статус-кодів  
- Доступ до старих UUID (№11, №12) — старі UUID не дають доступу до видалених профілів  
- Перевірка витоку даних через API після видалення (№6, №7, №11) — особисті дані та фото не повинні повертатися  

---

# Coverage Summary

| Area | Covered By |
|------|------------|
| Functional / CRUD | №2, №13–№16, №22 |
| Authentication / Login | №1, №4, №10, №21 |
| Delete logic / Delete | №3–№7, №11, №17, №18 |
| Role separation | №19–№22 |
| Data security / privacy | №6, №7, №11, №12 |
| Edge scenarios | №4, №5, №8–№10, №11, №18, №19–№22 |

---

# Key Findings
- Повне видалення акаунту працює коректно  
- Старі UUID більше недоступні  
- Повторна реєстрація з тим самим email створює новий акаунт з новою роллю та новим UUID  
- Витоку даних між ролями немає  
- Логін, активація та робота з профілями відбуваються коректно  

## Висновок:
E2E тестування підтвердило виправлення багу: публічний GET-запит не повертає видалені профілі.

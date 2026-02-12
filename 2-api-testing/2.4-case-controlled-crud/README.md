#  Case: Controlled CRUD Logic for Profile Statuses
**Project Type:** API Testing & Business Logic Validation  
**Role:** QA Engineer  
**Focus:** Data integrity, moderation workflow, REST compliance, cross-team collaboration
—
## TL;DR

- Виявлено критичний дефект: POST поводився як PUT, порушуючи REST-логіку та бізнес-правила.
- Дефект було виправлено.
- Під час подальшого тестування PUT виявлено системну проблему: відсутність контрольованих правил редагування профілів.
- Ініційовано міжкомандну дискусію (BE, FE, BA, QA) та запропоновано нову модель CRUD-логіки.
- Впроваджено контрольовані переходи статусів, що зменшило ризик неконсистентних даних та підвищило надійність API.

---
##  From Defect to Improvement

### Bug Phase

- Виявлено, що POST запит змінює вже існуючі профілі (фактично поводиться як PUT).
- Створено баг-репорт.
- Дефект виправлено відповідно до бізнес-логіки та REST-принципів.

### Improvement Phase
Результати тестування PUT продемонстрували потребу у системному покращенні.    
Було ініційовано міжкомандну дискусію (BE, FE, BA, QA) та запропоновано нові правила CRUD, які:
- забезпечують контрольоване створення та редагування профілів залежно від їх статусу;
- запобігають хаотичним змінам даних;
- гарантують відповідність бізнес-логіці та REST-архітектурі;
- покращують модераційний workflow.
---
## Key Outcomes

- Чітко розмежовано:
  - POST – лише для створення профілів;
  - PUT – лише для редагування дозволених статусів.
- Запроваджено валідацію допустимих переходів статусів.
- Заблоковано редагування профілів у статусах модерації або блокування.
- API повертає коректні HTTP status codes при порушенні правил.
- Знижено ризик неконсистентних даних.
- Підвищено передбачуваність та стабільність API-поведінки.


---
## Documentation Links
- [`Original Bug Report`](./bug-report.md)  
- [`Proposed Improvement`](./proposed-improvement.md)
---

## Visual Evidence 
Detailed screenshots are available in the `screenshots/` directory:
- `screenshots/bug/` – original defect reproduction
- `screenshots/improvement/` – before and after behavior following the improvement

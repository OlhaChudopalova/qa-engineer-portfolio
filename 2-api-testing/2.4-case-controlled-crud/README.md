#  Case: Controlled CRUD Logic for Profile Statuses

#### Project Type: API Testing & Business Logic Validation  
#### Role: QA Engineer  
#### Focus: Data integrity, moderation workflow, REST compliance, cross-team collaboration

---

## Goal

Уникнути хаотичного оновлення профілів, чітко розділити створення (POST) та редагування (PUT), а також забезпечити контрольовані переходи статусів відповідно до бізнес-логіки.

---

##  From Defect to Improvement


#### Bug Phase
- Виявлено, що POST працює як PUT та некоректно змінює статуси.  
- Дефект усунуто згідно з початковою пропозицією.

####  Improvement Phase
- Ініційовано та впроваджено нову систему правил CRUD із контрольованими статусами.
  
---

## Key Outcomes
- POST та PUT чітко розмежовані.
- Усі операції перевіряють статус профілю.
- Унеможливлено хаотичне оновлення даних.
- Підвищено надійність та передбачуваність API.
  
---

##  Details
- [`bug-report.md`](./bug-report.md)
- [`improvement-proposal.md`](./improvement-proposal.md)

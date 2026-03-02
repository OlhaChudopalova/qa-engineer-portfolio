#  Case: Controlled CRUD Logic for Profile Statuses

**Project Type:** API Testing & Business Logic Validation  
**Role:** QA Engineer  
**Focus:** Data integrity, moderation workflow, REST compliance, cross-team collaboration

---

## TL;DR

- A critical REST violation: the `POST` endpoint modified existing profiles instead of creating new ones.  
- Further testing revealed a systemic gap in profile status control.  
- A cross-team discussion (BE, FE, BA, QA), initiated to address this, resulted in controlled CRUD logic with validated status transitions.

---

## From Defect to System Improvement

### Bug Phase

- Identified that the POST request modified existing profiles (effectively behaving as PUT).
- Created a detailed bug report.
- The defect was fixed in accordance with business logic and REST architecture principles.

### Improvement Phase

Further PUT testing highlighted the need for systemic improvement.  

As a result, new CRUD rules were proposed and implemented to:

- Ensure controlled profile creation and editing based on profile status.  
- Prevent uncontrolled or chaotic data modifications.  
- Guarantee compliance with business logic and REST architecture.  
- Improve the moderation workflow.

---

## Key Outcomes

- Clear separation of responsibilities:
   - `POST` – strictly for profile creation;
   - `PUT` – strictly for editing profiles in allowed statuses.          
- Validation of permitted status transitions implemented.
- Editing blocked for profiles in moderation or blocked statuses.
- API now returns correct HTTP status codes when business rules are violated.
- Reduced risk of inconsistent data.
- Improved predictability and stability of API behavior.

---

## Documentation Links

- [`Original Bug Report`](./bug-report.md)  
- [`Proposed Improvement`](./proposed-improvement.md)
  
---

## Visual Evidence
Screenshots: [Original defect](screenshots/bug/) | [Improved behavior](screenshots/improvement/)


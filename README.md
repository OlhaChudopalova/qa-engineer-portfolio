# QA Portfolio

Hi! I’m Olha Chudopalova, a QA Engineer with experience in security-focused testing and cross-layer validation.  
This portfolio highlights my work on the **MVP project at Junfolio**, showcasing expertise in **Static Testing, API Testing, and Improvements**, with practical cases and security-focused analysis.

---

## Portfolio Structure

---

## [1. Static Testing](1-static-testing/README.md)
Static testing involves evaluating software artifacts without executing the code.
Focus areas include logic analysis, UX review, text/content evaluation, and detection of requirement ambiguities.

### [1.1. Text, Terminology, and UX Review](1-static-testing/1.1-text-terminology-and-ux-review/README.md)
- [1.1.1 Notification Text Review](1-static-testing/1.1-text-terminology-and-ux-review/1.1.1-notification-text-review.md)   
Analyzed system notification messages for clarity, tone, consistency, and user comprehension. Suggested improvements to make notifications more informative and actionable.

- [1.1.2 Home Page Text and UX Review](1-static-testing/1.1-text-terminology-and-ux-review/1.1.2-home-page-text-and-ux-review.md)  
Evaluated home page content, call-to-action wording, and overall user experience flow. Recommendations focused on enhancing usability, readability, and intuitive navigation.

### [1.2. Improving Password Validation](1-static-testing/1.2-improving-password-validation.md)  
Static analysis of password requirements, including security standards, specification clarity, and UX considerations. Addressed potential ambiguities, suggested microcopy improvements, and provided guidance for better password creation UX.

---

## [2. API Testing](2-api-testing/README.md)

This section contains tests and documentation related to API endpoints, including user profiles and other backend flows.

- [2.1. Bug: Invalid JSON Causes Data Loss](2-api-testing/2.1-bug-invalid-json-causes-data-loss/README.md)  
Uncovered server-side validation gaps where malformed JSON structures could overwrite fields, causing data loss and business rule violations.

- [2.2. Account Deletion Logic: Soft to Hard Delete](2-api-testing/2.2-account-deletion-logic-soft-to-hard-delete/README.md)  
Documented the transition from soft delete to hard delete, ensuring full user isolation, data privacy, and preventing cross-role data carryover issues.

- [2.3. API Returns Data of a Deleted Profile](2-api-testing/2.3-api-returns-data-of-a-deleted-profile/README.md)  
Revealed a critical issue where deleted profiles remained accessible via the API, creating a potential data leak.

- [2.4. Case-Controlled CRUD](2-api-testing/2.4-case-controlled-crud/README.md)  
Identified REST violations and implemented controlled CRUD logic with status-based transitions to ensure data integrity and predictable moderation workflow.

- [2.5. Rate Field Validation Alignment: API & UI](2-api-testing/2.5-rate-field-validation-alignment-api-ui/README.md)  
Examined cross-layer validation inconsistencies between API and UI and documented the alignment with business requirements after fixes.

---

## [3. Improvements](3-improvement/README.md)  

Proactive improvement proposals, security hardening initiatives, and system-level enhancement ideas aimed at improving platform reliability, data protection, and overall product quality.

- [3.1. Security: Preventing Email Enumeration in Password Reset Flow](3-improvement/3.1-security-email-enumeration-password-reset/README.md)
- [3.2. Clarification of Available / Unavailable Button Logic](3-improvement/3.2-available-unavailable-button-logic/README.md)

---

## Tools & Technologies Used in the Project

- **SWAGGER, REDOC**  
Analyzed API schemas and documentation, coordinating field validation and naming standards between Frontend and Backend.
- **POSTMAN**  
Performed API testing (CRUD operations, JSON structure analysis, environment variables, Bearer/JWT authorization, response and status code validation).
- **BROWSER DEVTOOLS**  
Conducted network analysis, request/response inspection, console error tracking, and frontend-backend integration checks.
- **JIRA**  
Used for bug reporting, improvement proposals, and project task tracking.
- **CONFLUENCE**  
Used for QA documentation and team coordination, including:  
  - Developed Test Plans (Strategy, Objectives).  
  - Created profile field validation documents for Frontend and Backend, resolving discrepancies and ensuring a unified standard.  
  - Built standardization tables for platform terminology.  
  - Created a document analyzing GET request security for user profiles and resolving discrepancies between Frontend and Backend.  
  - Documented email validation rules, coordinated with Frontend and Backend.
- **DJANGO ADMIN**  
Managing users, profiles, tokens, and access rights.
- **FIGMA**  
UI/UX review, layout consistency analysis, interaction behavior analysis, and content/terminology standardization.

---

## Key Strengths
- Cross-layer validation alignment (API ↔ UI)
- Business logic analysis
- Security-focused testing and risk identification
- Documentation-driven quality improvement

---

## Contacts
- LinkedIn: [olha-chudopalova](https://www.linkedin.com/in/olha-chudopalova-60a15776/)
- Email: [chudo1805@gmail.com](mailto:chudo1805@gmail.com)

---

## How to Use This Portfolio
- Select a category: Static Testing, API Testing, or Improvements.
- Open any case to explore the results of my work.


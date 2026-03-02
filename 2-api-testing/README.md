# API Testing

This folder contains tests and documentation related to API endpoints, including user profile, and other backend flows.  

## Test Cases

 
- [2.1 Bug: Invalid JSON Causes Data Loss](./2.1-bug-invalid-json-causes-data-loss) – uncovers server-side validation gaps where malformed JSON structure overwrites fields, causing data loss and business rule violations.
- [2.2 Account Deletion Logic: Soft to Hard Delete](./2.2-account-deletion-logic-soft-to-hard-delete) – documents the transition from soft delete to hard delete, eliminating identity reuse and cross-role data carryover, ensuring full user isolation and data privacy.
- [2.3 API Returns Data of a Deleted Profile](./2.3-api-returns-data-of-a-deleted-profile) – reveals a critical issue where deleted profiles remained accessible via the API, creating a potential data leak.
- [2.4 Case-Controlled CRUD](./2.4-case-controlled-crud) – identifies REST violations and implements controlled CRUD logic with status-based transitions to ensure data integrity and predictable moderation workflow.
- [2.5 Rate Field Validation Alignment: API & UI](./2.5-rate-field-validation-alignment-api-ui) – examines cross-layer validation inconsistencies between API and UI and documents the alignment with business requirements after fixes.

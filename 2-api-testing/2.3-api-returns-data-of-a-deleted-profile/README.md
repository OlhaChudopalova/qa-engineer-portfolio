# API Returns Data of a Deleted Profile

This folder contains the bug report, end-to-end (E2E) test results, and screenshots for a critical issue: the public API returned data of a deleted user profile.

## Issue:
 Deleted profiles were still accessible via the API, creating a potential data leak and violating the expected deletion logic.
 
## Fix:
 After the fix, deleted profiles are no longer accessible via the API, old UUIDs no longer work, and re-registration creates a new account with a clean profile.
 
## Contents:
- bug_report.md – Detailed bug report
- e2e_test_summary.md – E2E API test results
- screenshots/ – Screenshots of DELETE and GET requests

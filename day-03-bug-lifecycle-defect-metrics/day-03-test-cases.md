# Day 03 — Test Cases: Bug Lifecycle Validation + Advanced Scenarios
**Tester:** QA Engineer
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** Regression testing, security edge cases, data integrity validation

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 20 | 12 | 6 | 2 |

---

## Section 1 — Regression Tests: Previously Fixed Scenarios

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_051 | Regression | Login with valid credentials still works after security patch | Login with Admin/admin123 | Dashboard loads | Dashboard loaded | ✅ Pass |
| TC_052 | Regression | Empty field validation still works on login | Submit empty form | Validation shown | Validation shown | ✅ Pass |
| TC_053 | Regression | Leave request submission still works | Apply 2-day leave | Request created | Request created with Pending status | ✅ Pass |
| TC_054 | Regression | Employee creation still works | Add new employee with full details | Employee record created | Record created | ✅ Pass |

---

## Section 2 — Security Edge Cases

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_055 | Security | File upload accepts only images | Upload .exe file to profile photo | Rejected with error | File accepted — no validation | ⚠️ Fail |
| TC_056 | Security | Deleted user cannot login | Delete employee, try to login with their credentials | Login fails | Login succeeded — session created | ⚠️ Fail |
| TC_057 | Security | Session expires after inactivity | Leave app idle for 30 minutes, try to perform action | Redirected to login | Session remained active — no timeout observed | ⚠️ Fail |
| TC_058 | Security | Direct URL access without login | Navigate directly to /pim/viewEmployeeList without logging in | Redirected to login | Correctly redirected to login | ✅ Pass |
| TC_059 | Security | ESS user cannot access admin URLs | Login as ESS, navigate to /admin/viewAdminModule | Access blocked | Access blocked — redirected to dashboard | ✅ Pass |

---

## Section 3 — Data Integrity Validation

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_060 | Data Integrity | Duplicate Employee ID rejected | Create two employees with same ID: EMP999 | Second creation blocked | Both records created — duplicate allowed | ⚠️ Fail |
| TC_061 | Data Integrity | Employee ID field accepts only alphanumeric | Enter special chars in Employee ID: EMP@#99 | Rejected with validation | Accepted — no validation on format | ⚠️ Fail |
| TC_062 | Data Integrity | Negative leave balance displayed clearly | Approve excess leave — balance goes negative | Red warning shown | Negative balance shown with no visual indicator | ⚠️ Fail |
| TC_063 | Data Integrity | Deleted employee removed from all lists | Delete employee, check leave list and reports | Employee absent from all areas | Employee removed from PIM but historical leave records still show their name | ✅ Pass (acceptable behaviour) |

---

## Section 4 — UX and Usability Edge Cases

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_064 | UX | Search results clear when input deleted | Type in search, delete text | Full list restores | Filtered results remain — must click Search again | ⚠️ Fail |
| TC_065 | UX | Form preserves data on validation error | Fill long form, miss required field, submit | Data preserved, only error shown | Data preserved correctly | ✅ Pass |
| TC_066 | UX | Confirmation dialog before delete | Click delete on employee record | Confirmation dialog appears | "Are you sure?" dialog shown | ✅ Pass |
| TC_067 | UX | Loading indicator shown on slow actions | Perform search with large dataset | Loading spinner shown | Spinner shown during search | ✅ Pass |
| TC_068 | UX | Page title updates correctly on navigation | Navigate between modules | Browser tab title updates | Title updates correctly | ✅ Pass |
| TC_069 | UX | Browser back button works as expected | Navigate forward through app, press Back | Previous page loads | Previous page loaded correctly | ✅ Pass |

---

## Section 5 — Intermittent / Flaky Scenarios

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_070 | Flaky | Session timeout on tab switch | Open app, switch to another app for 20 mins, return | Session expired | Session still active — intermittent behaviour observed | 🚫 Blocked — needs further investigation |


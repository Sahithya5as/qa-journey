# Day 02 — Test Cases: OrangeHRM Leave Module + PIM
**Tester:** QA Engineer
**Date:** 2026-05-19
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Modules:** Leave Module, PIM (Employee Management)
**Techniques Applied:** Equivalence Partitioning, Boundary Value Analysis, Decision Table, State Transition, Use Case Testing

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 25 | 18 | 5 | 2 |

---

## Section 1 — Equivalence Partitioning: Leave Comment Field

| TC ID | Technique | Scenario | Steps | Test Data | Expected Result | Actual Result | Status |
|-------|-----------|----------|-------|-----------|-----------------|---------------|--------|
| TC_021 | EP — Valid | Normal text in comment field | 1. Go to Leave → Apply Leave 2. Enter normal comment 3. Submit | "Attending a family function" | Comment accepted, leave submitted | Accepted successfully | ✅ Pass |
| TC_022 | EP — Valid min | Single character in comment | 1. Go to Apply Leave 2. Enter one character 3. Submit | "A" | Accepted | Accepted | ✅ Pass |
| TC_023 | EP — Invalid empty | Empty comment when required | 1. Go to Apply Leave 2. Leave comment empty 3. Submit | (empty) | Validation error shown | No error shown — comment not mandatory | ✅ Pass (field is optional) |
| TC_024 | EP — Invalid over limit | Extremely long comment | 1. Go to Apply Leave 2. Paste 1000 character string 3. Submit | 1000 chars | Error or character limit shown | Field accepted all 1000 chars — no frontend limit | ⚠️ Fail |

---

## Section 2 — Boundary Value Analysis: Leave Date Range

| TC ID | Technique | Scenario | Steps | Test Data | Expected Result | Actual Result | Status |
|-------|-----------|----------|-------|-----------|-----------------|---------------|--------|
| TC_025 | BVA — Below min | Past date as start date | 1. Apply Leave 2. Set start date to yesterday | Yesterday's date | Rejected — cannot apply for past leave | Error shown "Date should be after today" | ✅ Pass |
| TC_026 | BVA — At min | Today as start date | 1. Apply Leave 2. Set start date to today | Today's date | Accepted | Accepted | ✅ Pass |
| TC_027 | BVA — Above min | Tomorrow as start date | 1. Apply Leave 2. Set start date to tomorrow | Tomorrow's date | Accepted | Accepted | ✅ Pass |
| TC_028 | BVA — Same day | Start and end date same day | 1. Apply Leave 2. Set start = end = tomorrow | Same date both fields | Accepted as single day leave | Accepted — half day option shown | ✅ Pass |
| TC_029 | BVA — Invalid range | End date before start date | 1. Apply Leave 2. Set end date before start date | Start: next Monday, End: last Friday | Error — end date cannot be before start date | Error shown but message just says "Invalid" — not specific enough | ⚠️ Fail |
| TC_030 | BVA — Far future | Date 1 year in future | 1. Apply Leave 2. Set date 365 days ahead | 1 year from today | Accepted or system shows future limit | Accepted with no limit — no maximum future date validation | ⚠️ Fail |

---

## Section 3 — Decision Table: Leave Approval Logic

| TC ID | Technique | Scenario | Steps | Test Data | Expected Result | Actual Result | Status |
|-------|-----------|----------|-------|-----------|-----------------|---------------|--------|
| TC_031 | Decision Table | All conditions met — approve succeeds | 1. Employee applies leave 2. Manager approves 3. Balance sufficient | Valid request, sufficient balance | Leave approved successfully | Approved successfully | ✅ Pass |
| TC_032 | Decision Table | Insufficient leave balance | 1. Set employee balance to 0 2. Employee applies leave 3. Manager approves | Zero balance, leave requested | Warning shown — balance insufficient | No warning — leave approved with zero balance | ⚠️ Fail |
| TC_033 | Decision Table | No approval rights | 1. Login as ESS employee 2. Try to approve another employee's leave | ESS user, another user's leave | Approve option not available | Approve button not shown — correctly blocked | ✅ Pass |
| TC_034 | Decision Table | No pending request | 1. Login as Admin 2. Check leave list with no pending requests | No pending requests exist | No approve action available | No requests shown — correct | ✅ Pass |

---

## Section 4 — State Transition: Leave Request Workflow

| TC ID | Technique | Start State | Action | Expected End State | Actual Result | Status |
|-------|-----------|-------------|--------|--------------------|---------------|--------|
| TC_035 | State Transition | Pending | Manager approves | Approved | Status changed to Approved | ✅ Pass |
| TC_036 | State Transition | Pending | Manager rejects | Rejected | Status changed to Rejected | ✅ Pass |
| TC_037 | State Transition | Pending | Employee cancels | Cancelled | Status changed to Cancelled | ✅ Pass |
| TC_038 | State Transition | Approved | Employee cancels before leave date | Cancelled | Status changed to Cancelled | ✅ Pass |
| TC_039 | State Transition | Rejected | Attempt to approve | Should be blocked | Approve option not available — correctly blocked | ✅ Pass |
| TC_040 | State Transition | Cancelled | Attempt to approve | Should show error | Button did nothing — no error message shown to user | ⚠️ Fail |

---

## Section 5 — Use Case Test: Full Employee Onboarding Journey

| TC ID | Technique | Step | Action | Expected Result | Actual Result | Status |
|-------|-----------|------|--------|-----------------|---------------|--------|
| TC_041 | Use Case | 1 | Login as Admin | Dashboard loads | Dashboard loaded | ✅ Pass |
| TC_042 | Use Case | 2 | Go to PIM → Add Employee | Form opens | Form opened | ✅ Pass |
| TC_043 | Use Case | 3 | Fill name: Test QAEngineer, ID: TQA001 | Fields accept input | Input accepted | ✅ Pass |
| TC_044 | Use Case | 4 | Enable login: testqa / Test@1234 | Login details saved | Saved | ✅ Pass |
| TC_045 | Use Case | 5 | Save employee record | Employee created | Record created | ✅ Pass |
| TC_046 | Use Case | 6 | Logout as Admin | Redirected to login | Redirected | ✅ Pass |
| TC_047 | Use Case | 7 | Login as testqa | Employee dashboard loads | Limited dashboard shown | ✅ Pass |
| TC_048 | Use Case | 8 | View own profile | Correct details shown | Name and ID correct | ✅ Pass |
| TC_049 | Use Case | 9 | Try to access Admin menu | Should be blocked | Admin menu not visible | ✅ Pass |
| TC_050 | Use Case | 10 | Admin deletes test employee | Record removed — cleanup | Deleted successfully | ✅ Pass |

---

## Key Observations

1. **Leave balance not enforced** — Manager could approve leave when employee had zero remaining balance. No warning or block. Major business logic gap.
2. **No maximum future date** — Leave can be applied 1 year or more in advance with no system limit.
3. **Vague validation messages** — "Invalid" is not helpful. Messages should explain exactly what is wrong.
4. **Silent failure on invalid state transition** — Trying to approve a cancelled leave did nothing with no feedback. User has no idea if their action was processed or ignored.
5. **Comment field has no character limit** — Accepts unlimited text with no frontend validation.


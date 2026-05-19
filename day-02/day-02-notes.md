# Day 02 — Test Design Techniques
**Date:** 2026-05-19
**Phase:** 1 — Foundations
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com
**Module Tested:** Leave Module + PIM (Employee Management)

---

## What I Learned Today

Test design techniques are systematic methods for writing test cases that maximise coverage while minimising the number of tests needed. Without techniques, test cases are written randomly and important scenarios get missed. With techniques, every test case has a reason and nothing is left to chance.

Today I studied and applied 5 core techniques that every senior QA engineer uses daily.

---

## Technique 1 — Equivalence Partitioning (EP)

### Concept
Divide all possible inputs into groups called partitions. Every value inside one partition behaves the same way — so you only need to test one value from each group. This dramatically reduces the number of test cases without reducing coverage.

### How to identify partitions
- Valid partition — inputs the system should accept
- Invalid partition — inputs the system should reject
- There can be multiple valid and multiple invalid partitions

### Example applied — OrangeHRM Leave Comment field
The comment field on the Apply Leave page accepts free text.

| Partition | Description | Test Value | Expected |
|-----------|-------------|------------|----------|
| Valid — normal text | Text within allowed length | "Attending a family function" | Accepted |
| Valid — minimum | Single character | "A" | Accepted |
| Invalid — empty | No text when field is required | (empty) | Validation error shown |
| Invalid — over limit | Text exceeding character limit | 1000 character string | Error or truncation |

### Key interview point
"I use EP to reduce test cases without reducing coverage. Instead of testing every possible input, I identify all valid and invalid groups and test one representative value from each."

---

## Technique 2 — Boundary Value Analysis (BVA)

### Concept
Most bugs occur at the edges of valid ranges, not in the middle. BVA focuses testing effort on boundary values — the value just below the boundary, exactly at the boundary, and just above the boundary.

### Always test these 6 values for any range
For a range of Min to Max:
- Min - 1 (just below lower boundary) — invalid
- Min (at lower boundary) — valid
- Min + 1 (just above lower boundary) — valid
- Max - 1 (just below upper boundary) — valid
- Max (at upper boundary) — valid
- Max + 1 (just above upper boundary) — invalid

### Example applied — OrangeHRM Leave Date Range
Leave requests have a date range. The boundary conditions are around today's date.

| Test Value | Boundary Type | Expected Result |
|------------|---------------|-----------------|
| Yesterday's date as start date | Below lower boundary | Should be rejected — cannot apply for past leave |
| Today's date | At lower boundary | Should be accepted |
| Tomorrow's date | Above lower boundary | Should be accepted |
| Start date = End date | Same day leave | Should be accepted — half day or full day |
| End date before Start date | Invalid range | Error message — end date cannot be before start date |
| Date 1 year in future | Far future | Should check if system has a future date limit |

### Finding from testing
When end date was set before start date, OrangeHRM showed a validation error. However the error message was not specific enough — it said "Invalid" without explaining which dates were wrong. Raised as BUG-006.

### Key interview point
"I always combine BVA with EP. EP tells me the partitions, BVA tells me to test the edges of those partitions where defects are most likely to hide."

---

## Technique 3 — Decision Table Testing

### Concept
Used when a feature has multiple input conditions where different combinations of inputs produce different outcomes. A decision table maps every possible combination of conditions to its expected result. Nothing gets missed because every combination is explicitly listed.

### When to use
- Features with multiple yes/no conditions
- Business rules with multiple inputs (discounts, access control, approvals)
- Any feature where "it depends on multiple things"

### Example applied — OrangeHRM Leave Approval

Conditions that affect whether leave can be approved:
- Does a pending leave request exist?
- Does the manager have approval rights?
- Does the employee have sufficient leave balance?

| Condition | TC1 | TC2 | TC3 | TC4 |
|-----------|-----|-----|-----|-----|
| Leave request exists | Yes | Yes | Yes | No |
| Manager has approval rights | Yes | Yes | No | Yes |
| Leave balance sufficient | Yes | No | Yes | Yes |
| **Expected Result** | **Approve succeeds** | **Warning shown** | **Approve blocked** | **No action available** |

Each column is one test case. 4 columns = 4 test cases. Every meaningful combination is covered.

### Finding from testing
TC2 — when leave balance was insufficient, the system allowed the manager to approve the leave without any warning. The leave was approved even though the employee had zero remaining balance. Raised as BUG-007 — critical business logic failure.

### Key interview point
"I use decision tables for features with complex business rules. They make it impossible to miss combinations and serve as living documentation of the business logic."

---

## Technique 4 — State Transition Testing

### Concept
Used when the system moves through different states based on user actions or events. You test every valid transition (allowed state change) and every invalid transition (state change that should be blocked).

### How to approach it
1. Identify all possible states the feature can be in
2. Identify all events that cause state changes
3. Draw a state diagram
4. Write test cases for every transition — valid and invalid

### Example applied — OrangeHRM Leave Request States

```
States:       Pending → Approved → (Leave taken)
              Pending → Rejected
              Pending → Cancelled (by employee)
              Approved → Cancelled (before leave date)
```

| TC | Starting State | Action | Expected End State |
|----|---------------|--------|--------------------|
| ST_001 | Pending | Manager approves | Approved |
| ST_002 | Pending | Manager rejects | Rejected |
| ST_003 | Pending | Employee cancels | Cancelled |
| ST_004 | Approved | Employee cancels before leave date | Cancelled |
| ST_005 | Rejected | Employee tries to approve | Should be blocked — invalid transition |
| ST_006 | Cancelled | Manager tries to approve | Should be blocked — invalid transition |

### Finding from testing
ST_005 — attempting to approve a rejected leave request was correctly blocked by the system. However ST_006 — trying to approve a cancelled request showed no error message, the button simply did nothing with no feedback to the user. Raised as BUG-008 — missing user feedback on invalid state transition.

### Key interview point
"State transition testing is essential for workflow-based features. I always test both the valid paths and the invalid transitions — trying to perform actions that should not be allowed is where many bugs hide."

---

## Technique 5 — Use Case Testing

### Concept
Tests the complete end-to-end user journey from start to finish, exactly as a real user would interact with the system. Instead of testing individual fields or buttons in isolation, use case testing validates that the entire workflow works together correctly.

### Why it matters
Individual unit tests can all pass while the integrated flow completely fails. Use case testing catches integration bugs that isolated tests miss.

### Example applied — Full Employee Onboarding Journey

**Use Case: HR Admin adds a new employee who then logs in and views their profile**

| Step | Action | Expected Result | Actual Result | Status |
|------|--------|-----------------|---------------|--------|
| 1 | Login as Admin | Dashboard loads | Dashboard loaded | ✅ Pass |
| 2 | Go to PIM → Add Employee | Add employee form opens | Form opened | ✅ Pass |
| 3 | Fill in name: Test QAEngineer, ID: TQA001 | Fields accept input | Fields accepted input | ✅ Pass |
| 4 | Enable login: set username testqa / password Test@1234 | Login details saved | Saved successfully | ✅ Pass |
| 5 | Click Save | Employee record created | Record created, redirected to employee profile | ✅ Pass |
| 6 | Log out as Admin | Redirected to login page | Redirected correctly | ✅ Pass |
| 7 | Login as testqa / Test@1234 | Employee dashboard loads | Dashboard loaded with limited menu | ✅ Pass |
| 8 | View own profile under My Info | Profile shows correct details | Correct name and ID shown | ✅ Pass |
| 9 | Try to access Admin menu | Should be blocked | Admin menu not visible — access correctly restricted | ✅ Pass |
| 10 | Login as Admin, delete test employee | Record removed | Employee deleted — cleanup complete | ✅ Pass |

### Key insight
Step 9 is an access control test embedded inside a use case test. Real-world QA often combines multiple test types in one end-to-end scenario.

### Key interview point
"I always include at least one end-to-end use case test for every major feature. It validates that all the pieces work together, not just in isolation. I also include cleanup steps to leave the test environment in a clean state."

---

## Summary — When to Use Each Technique

| Technique | Use When |
|-----------|----------|
| Equivalence Partitioning | Input fields with ranges or categories |
| Boundary Value Analysis | Any numeric range or date range |
| Decision Table | Multiple conditions producing different outcomes |
| State Transition | Features with workflow states (pending, approved, etc.) |
| Use Case Testing | End-to-end user journeys, integration flows |

---

## Interview Talking Points from Today

- Can explain all 5 test design techniques with real examples from OrangeHRM
- Know when to use each technique — not just what they are but why
- Applied decision tables to leave approval business logic
- Applied state transition testing to leave request workflow
- Executed a complete end-to-end use case test including access control validation and test data cleanup
- Found 3 bugs using systematic techniques that random testing would likely have missed

---

## Bugs Found Today

| Bug ID | Title | Technique That Found It | Severity | Priority |
|--------|-------|------------------------|----------|----------|
| BUG-006 | Leave application — error message not specific when end date is before start date | BVA | Low | Low |
| BUG-007 | Leave approval — manager can approve leave when employee has zero balance, no warning shown | Decision Table | High | High |
| BUG-008 | Leave request — no feedback shown when manager tries to approve a cancelled request | State Transition | Medium | Medium |


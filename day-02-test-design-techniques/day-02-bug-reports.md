# Day 02 — Bug Reports: OrangeHRM Leave Module
**Tester:** QA Engineer
**Date:** 2026-05-19
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com

---

## Summary

| Bug ID | Title | Severity | Priority | Found By |
|--------|-------|----------|----------|----------|
| BUG-006 | End date before start date shows vague "Invalid" error with no specific guidance | Low | Low | BVA |
| BUG-007 | Manager can approve leave when employee has zero balance — no warning or block | High | High | Decision Table |
| BUG-008 | No user feedback when manager attempts to approve a cancelled leave request | Medium | Medium | State Transition |

---

## BUG-006

**Title:** Leave Application — Vague "Invalid" error shown when end date is set before start date — no specific guidance given — Chrome v124 / macOS

**Severity:** Low
**Priority:** Low
**Type:** UX / Validation
**Found Using:** Boundary Value Analysis

**Steps to Reproduce:**
1. Login as Admin at https://opensource-demo.orangehrmlive.com
2. Go to Leave → Apply Leave
3. Set Start Date to next Monday
4. Set End Date to last Friday (a date before the start date)
5. Click Apply

**Expected Result:**
A clear, specific error message should be shown, for example:
"End date cannot be before start date. Please select a valid date range."

**Actual Result:**
A generic "Invalid" message is displayed. The user cannot tell from this message which field is wrong or what needs to be corrected.

**Impact:**
Low — the validation does work and the leave cannot be submitted. However the user experience is poor and will lead to confusion, especially for first-time users.

**Suggested Fix:**
Replace the generic "Invalid" message with specific guidance:
"The end date must be on or after the start date."

---

## BUG-007

**Title:** Leave Module — Manager can approve leave request when employee has zero remaining leave balance — no warning or blocking shown — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Business Logic / Functional
**Found Using:** Decision Table Testing

**Steps to Reproduce:**
1. Login as Admin
2. Go to Leave → Leave Entitlements — set an employee's leave balance to 0 days
3. Login as that employee (or apply leave on their behalf as Admin)
4. Apply for 2 days of leave
5. Login as Admin → Go to Leave → Leave List
6. Find the pending leave request
7. Click Approve

**Expected Result:**
The system should either:
- Block the approval and show: "Cannot approve — employee has insufficient leave balance (0 days remaining)"
- OR show a warning before approving: "This employee has 0 days remaining. Approving will result in a negative balance. Do you want to continue?"

**Actual Result:**
The leave is approved without any warning or error. The employee's leave balance goes into negative. No indication is given to the manager that the balance is insufficient.

**Impact:**
High — this is a core business logic failure. In a real company, approving leave without checking balance leads to payroll errors, HR policy violations, and disputes. This would be flagged as a critical issue in any production HR system.

**Evidence:**
Confirmed by setting balance to 0, submitting leave, and approving — leave status changed to Approved with balance showing -2 days.

---

## BUG-008

**Title:** Leave Module — No error message or feedback shown when manager attempts to approve a cancelled leave request — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** UX / State Management
**Found Using:** State Transition Testing

**Steps to Reproduce:**
1. Login as Admin
2. Go to Leave → Apply Leave — submit a leave request as an employee
3. Go to Leave → My Leave List — cancel the leave request (status changes to Cancelled)
4. Login as Admin → Go to Leave → Leave List
5. Find the cancelled leave request
6. Attempt to click Approve or any action button on the cancelled request

**Expected Result:**
Either:
- The approve option should not be visible for cancelled requests
- OR if the button is visible and clicked, show a clear message: "This leave request has been cancelled and cannot be approved."

**Actual Result:**
The action button is present but clicking it produces no response — no success message, no error message, no state change. The user has no idea whether their action was processed, ignored, or failed. This is a silent failure.

**Impact:**
Medium — a manager attempting to take action on a cancelled request is left completely without feedback. This creates confusion about whether the system is working correctly and whether the action needs to be retried.

**Suggested Fix:**
Either hide the approve button for cancelled requests, or show a clear message explaining the request is cancelled and no further action is possible.


# Day 04 — Bug Reports: Risk-Based Testing Findings
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** High risk areas — Leave approval and Authentication

---

## Summary

| Bug ID | Title | Severity | Priority | Found By |
|--------|-------|----------|----------|----------|
| BUG-014 | Overlapping leave requests accepted for same dates | High | High | Risk-based — Leave module |
| BUG-015 | Concurrent sessions allowed for same user account | Medium | Medium | Risk-based — Authentication |
| BUG-016 | Orphaned leave records remain after employee deletion | Medium | Low | Shift-left requirement gap |

---

## BUG-014

**Title:** Leave Module — System accepts two overlapping leave requests for the same employee on the same dates — no duplicate detection — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Business Logic
**Found By:** Risk-based testing — Leave module identified as high risk

**Steps to Reproduce:**
1. Login as Admin
2. Go to Leave → Apply Leave
3. Select any employee, set dates: next Monday to Wednesday
4. Submit — status becomes Pending
5. Go to Leave → Apply Leave again
6. Select the same employee, set the exact same dates
7. Submit again

**Expected Result:**
Second request rejected: "A leave request already exists for the selected dates."

**Actual Result:**
Both requests created as Pending. Both can be independently approved — employee would have double leave recorded for the same period.

**Impact:**
Double leave deduction from balance, incorrect payroll calculations, HR compliance violations.

---

## BUG-015

**Title:** Authentication — Same user account active in multiple browsers simultaneously — no session limit enforced — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** Security
**Found By:** Risk-based testing — Authentication full coverage

**Steps to Reproduce:**
1. Login with Admin/admin123 in Chrome
2. Login with same credentials in Firefox
3. Return to Chrome — session still active

**Expected Result:**
Second login should invalidate the first session OR show: "You are already logged in on another device."

**Actual Result:**
Both sessions fully active simultaneously. Same admin account operational in two browsers at once. No notification or restriction applied.

**Impact:**
Actions cannot be attributed to a specific person if multiple sessions are active — audit trail unreliable. Former employees or shared accounts pose a security risk.

---

## BUG-016

**Title:** PIM — Leave requests remain as orphaned records after associated employee is deleted — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Low
**Type:** Data Integrity
**Found By:** Shift-left requirement gap SL_004 confirmed through testing

**Steps to Reproduce:**
1. Create employee: Test Orphan, ID: ORP001
2. Apply a leave request for this employee — leave as Pending
3. Delete the employee from PIM
4. Go to Leave → Leave List — search for the deleted employee's leave

**Expected Result:**
Leave requests archived as "Former Employee" or deleted with the employee record.

**Actual Result:**
Leave request remains in the Leave List with the employee name shown but no active employee linked. Can still be approved or rejected.

**Root Cause:**
User story for employee deletion had no acceptance criteria covering related records. Flagged as SL_004 during shift-left requirement review — gap in requirements led directly to incomplete implementation.

**Impact:**
Orphaned records corrupt reporting accuracy. Leave reports and audit logs show data for non-existent employees.


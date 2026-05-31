# Day 03 — Bug Reports: OrangeHRM — Advanced Bug Lifecycle Testing
**Tester:** QA Engineer
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** Validating bug states, regression scenarios, and edge case defects

---

## Summary

| Bug ID | Title | Severity | Priority | Type | Status |
|--------|-------|----------|----------|------|--------|
| BUG-009 | Employee profile photo upload — no file type validation | High | High | Security / Functional | Open |
| BUG-010 | Leave balance displays negative values with no visual indicator | Medium | Medium | UX / Business Logic | Open |
| BUG-011 | Deleted employee username still accepted on login page — session not invalidated | High | High | Security | Open |
| BUG-012 | Search results not cleared when search term is deleted from field | Low | Low | UX | Open |
| BUG-013 | System allows duplicate employee ID to be created — no uniqueness validation | High | High | Data Integrity | Open |

---

## BUG-009

**Title:** PIM Module — Employee profile photo upload accepts all file types — no file type or file size validation — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Security / Functional

**Steps to Reproduce:**
1. Login as Admin
2. Go to PIM → Employee List → select any employee
3. Click on the profile photo area
4. Attempt to upload a non-image file (e.g. .pdf, .txt, .exe)
5. Observe the result

**Expected Result:**
Only image files should be accepted (JPG, PNG, GIF). Non-image files should be rejected with a message: "Only image files (JPG, PNG, GIF) are allowed. Maximum size: 2MB."

**Actual Result:**
Non-image files can be selected and uploaded without any client-side validation. No error message shown at the point of file selection.

**Impact:**
High — accepting executable files through file upload is a known attack vector. This represents a potential security vulnerability that could be exploited to upload malicious files to the server.

---

## BUG-010

**Title:** Leave Module — Employee leave balance shows negative values after excess leave approved — no visual warning or indicator shown — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** UX / Business Logic

**Steps to Reproduce:**
1. Login as Admin
2. Set an employee's annual leave balance to 1 day
3. Apply 3 days of leave for that employee
4. Approve the leave request
5. Go to Leave → Leave Entitlements — view the employee's balance

**Expected Result:**
Balance should show 0 (not go negative), OR a warning should be shown in red: "Balance: -2 days (Overdrawn)" with a visual indicator (red colour, warning icon).

**Actual Result:**
Balance shows -2 with no visual differentiation from a positive balance. No colour change, no icon, no warning. A manager reviewing balances would not immediately notice an employee is in deficit.

---

## BUG-011

**Title:** Login — Deleted employee credentials still accepted and session created — authentication not invalidated on deletion — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Security

**Steps to Reproduce:**
1. Login as Admin
2. Create a new employee with login credentials: username testdelete / password Test@1234
3. Note the employee record is active and login works
4. While logged in as Admin, delete this employee from PIM
5. Open a new browser tab
6. Attempt to login with testdelete / Test@1234

**Expected Result:**
Login should fail immediately after employee deletion. The system should invalidate all credentials associated with a deleted user account.

**Actual Result:**
Login succeeds using the deleted employee's credentials. A session is created and the user can access the system even though their employee record no longer exists.

**Impact:**
High — this is a security vulnerability. Terminated employees whose accounts are deleted should immediately lose all system access. In an HR system, this could allow a former employee to access sensitive company and personnel data after their employment ends.

---

## BUG-012

**Title:** PIM Employee Search — Search results not cleared when search input is manually deleted from the field — Chrome v124 / macOS

**Severity:** Low
**Priority:** Low
**Type:** UX

**Steps to Reproduce:**
1. Login as Admin
2. Go to PIM → Employee List
3. Type "test" in the search field — search results filter correctly
4. Manually delete the text from the search field using backspace
5. Observe the employee list

**Expected Result:**
When the search field is cleared, the full employee list should restore automatically.

**Actual Result:**
The filtered results remain visible even after the search field is emptied. The user must click the Search button again or refresh the page to see the full list.

**Impact:**
Low — cosmetic UX issue. No data loss or functional impact.

---

## BUG-013

**Title:** PIM — System allows creation of two employees with identical Employee ID — no uniqueness validation — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Data Integrity

**Steps to Reproduce:**
1. Login as Admin
2. Go to PIM → Add Employee
3. Enter First Name: Test, Last Name: One, Employee ID: EMP999
4. Save the record
5. Go to PIM → Add Employee again
6. Enter First Name: Test, Last Name: Two, Employee ID: EMP999 (same ID)
7. Save the record

**Expected Result:**
The system should reject the duplicate Employee ID and show: "Employee ID EMP999 already exists. Please use a unique ID."

**Actual Result:**
Both employee records are created successfully with the same Employee ID. No validation error shown.

**Impact:**
High — duplicate employee IDs corrupt data integrity in an HR system. Payroll processing, leave calculations, and reporting all depend on Employee ID being a unique identifier. This could cause incorrect salary payments, leave allocation errors, and compliance issues.


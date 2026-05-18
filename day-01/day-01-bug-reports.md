# Day 01 — Bug Reports: OrangeHRM Login Module
**Tester:** QA Engineer
**Date:** 2026-05-18
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com

---

## Summary

| Bug ID | Title | Severity | Priority | Status |
|--------|-------|----------|----------|--------|
| BUG-001 | No account lockout after 5 failed login attempts | High | High | Open |
| BUG-002 | Username field accepts unlimited characters — no frontend validation | Low | Medium | Open |
| BUG-003 | Leading whitespace in username causes silent login failure | Medium | Medium | Open |
| BUG-004 | Username is case-sensitive with no user-facing indication | Medium | Low | Open |
| BUG-005 | Double-clicking Login button fires two simultaneous POST requests | Medium | Medium | Open |

---

## BUG-001

**Title:** Login page — No account lockout or CAPTCHA after 5 consecutive failed login attempts — Security vulnerability — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Security

**Steps to Reproduce:**
1. Open https://opensource-demo.orangehrmlive.com
2. Enter username: Admin
3. Enter wrong password: wrongpassword
4. Click Login
5. Repeat steps 2–4 five times with different wrong passwords

**Expected Result:**
After 3–5 failed attempts, system should lock the account OR show a CAPTCHA OR introduce a time delay.

**Actual Result:**
Login form remains fully accessible after 5 failed attempts. No lockout, no CAPTCHA, no delay, no warning.

**Impact:**
Allows unlimited brute force password attempts against any user account.

---

## BUG-002

**Title:** Login page — Username field accepts unlimited characters with no frontend length validation — Chrome v124 / macOS

**Severity:** Low
**Priority:** Medium
**Type:** Functional / UI

**Steps to Reproduce:**
1. Open https://opensource-demo.orangehrmlive.com
2. Paste a 300-character string into the username field
3. Observe all 300 characters are accepted
4. Click Login

**Expected Result:**
Field should have a maxlength limit (e.g. 50 chars). User should be stopped or warned if they exceed it.

**Actual Result:**
All 300 characters accepted with no frontend validation. Only the backend rejects it.

---

## BUG-003

**Title:** Login page — Leading whitespace in username field not trimmed — causes silent login failure with no guidance — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** Functional / UX

**Steps to Reproduce:**
1. Open https://opensource-demo.orangehrmlive.com
2. Type a space before the username: " Admin" (space + Admin)
3. Enter correct password: admin123
4. Click Login

**Expected Result:**
System trims whitespace and logs in successfully, OR shows a message: "Please check for extra spaces in your username."

**Actual Result:**
Login fails with generic "Invalid credentials." User has no way to know whitespace is the cause.

**Real-world impact:**
Users copying usernames from emails often accidentally include whitespace. This creates confusing failures.

---

## BUG-004

**Title:** Login page — Username field is case-sensitive with no user-facing indication — ADMIN fails silently — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Low
**Type:** UX / Functional

**Steps to Reproduce:**
1. Open https://opensource-demo.orangehrmlive.com
2. Enter username in all caps: ADMIN
3. Enter correct password: admin123
4. Click Login

**Expected Result:**
Either (a) system accepts ADMIN, Admin, and admin as equivalent, OR (b) login page shows "Username is case-sensitive."

**Actual Result:**
Login fails with "Invalid credentials." No indication that case sensitivity is the cause.

---

## BUG-005

**Title:** Login page — Double-clicking Login button fires two simultaneous POST requests to auth endpoint — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** Functional

**Steps to Reproduce:**
1. Open https://opensource-demo.orangehrmlive.com
2. Open Chrome DevTools → Network tab → tick Preserve Log
3. Enter username: Admin, password: admin123
4. Double-click the Login button rapidly
5. Observe the Network tab

**Expected Result:**
Only one POST request sent. Login button disabled immediately on first click.

**Actual Result:**
Two POST requests visible in Network tab fired within milliseconds of each other. Button not disabled after first click.

**Evidence:**
Confirmed by observing two POST requests to the auth endpoint in Chrome DevTools Network tab.

**Fix:**
Disable Login button on first click. Re-enable only if authentication fails.


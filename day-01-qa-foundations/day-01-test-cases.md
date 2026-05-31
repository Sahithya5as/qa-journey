# Day 01 — Test Cases: OrangeHRM Login Module
**Tester:** QA Engineer
**Date:** 2026-05-18
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Module:** Login / Authentication

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 20 | 14 | 5 | 1 |

---

## Test Cases

| TC ID | Scenario | Steps | Test Data | Expected Result | Actual Result | Status |
|-------|----------|-------|-----------|-----------------|---------------|--------|
| TC_001 | Valid login | 1. Open URL 2. Enter username 3. Enter password 4. Click Login | Admin / admin123 | Dashboard loads | Dashboard loaded successfully | ✅ Pass |
| TC_002 | Wrong password | 1. Open URL 2. Enter valid username 3. Enter wrong password 4. Click Login | Admin / wrongpass | Error message shown | "Invalid credentials" shown | ✅ Pass |
| TC_003 | Wrong username | 1. Open URL 2. Enter wrong username 3. Enter valid password 4. Click Login | wronguser / admin123 | Error message shown | Error shown, login blocked | ✅ Pass |
| TC_004 | Both fields empty | 1. Open URL 2. Leave both empty 3. Click Login | empty / empty | Validation on both fields | Validation shown on username only, not password | ⚠️ Fail |
| TC_005 | Username filled, password empty | 1. Open URL 2. Enter username 3. Leave password empty 4. Click Login | Admin / empty | Validation on password field | "Required" shown on password | ✅ Pass |
| TC_006 | Password filled, username empty | 1. Open URL 2. Leave username empty 3. Enter password 4. Click Login | empty / admin123 | Validation on username field | "Required" shown on username | ✅ Pass |
| TC_007 | Leading space in username | 1. Open URL 2. Enter " Admin" (space before) 3. Enter password 4. Click Login | " Admin" / admin123 | System trims space and logs in OR shows clear error | Login FAILED with "Invalid credentials" — no indication whitespace is the cause | ⚠️ Fail |
| TC_008 | Username in ALL CAPS | 1. Open URL 2. Enter ADMIN 3. Enter password 4. Click Login | ADMIN / admin123 | Logs in (case-insensitive) OR shows message about case | Login FAILED — system is case-sensitive with no user guidance | ⚠️ Fail |
| TC_009 | Very long username (300 chars) | 1. Open URL 2. Paste 300 character string 3. Enter password 4. Click Login | 300 chars / admin123 | Field limits input OR clean error shown | Field accepted all 300 chars — no frontend length validation | ⚠️ Fail |
| TC_010 | SQL injection in username | 1. Open URL 2. Enter ' OR '1'='1 3. Enter any password 4. Click Login | ' OR '1'='1 / anything | Normal error, no DB error exposed | "Invalid credentials" shown — handled correctly | ✅ Pass |
| TC_011 | Special characters in password | 1. Open URL 2. Enter username 3. Enter !@#$%^&*() 4. Click Login | Admin / !@#$%^&*() | Normal error shown, no crash | "Invalid credentials" shown cleanly | ✅ Pass |
| TC_012 | Press Enter key to submit | 1. Open URL 2. Enter credentials 3. Press Enter key | Admin / admin123 | Form submits, login succeeds | Login succeeded on Enter key | ✅ Pass |
| TC_013 | Double-click Login button | 1. Open URL 2. Enter credentials 3. Double-click Login fast | Admin / admin123 | One request sent, one login | Two POST requests fired (seen in DevTools Network tab) | ⚠️ Fail |
| TC_014 | Copy-paste password | 1. Copy "admin123" 2. Paste into password field 3. Click Login | Admin / admin123 (pasted) | Login succeeds | Login succeeded — copy-paste works | ✅ Pass |
| TC_015 | 5 failed attempts — lockout check | 1. Enter wrong credentials 2. Click Login 3. Repeat 5 times | Admin / wrongpass x5 | Account locks or CAPTCHA appears | No lockout after 5 attempts — security concern | ⚠️ Fail |
| TC_016 | Forgot Password link | 1. Open URL 2. Click "Forgot your password?" | N/A | Reset page opens | Password reset page loaded successfully | ✅ Pass |
| TC_017 | Browser Back after login | 1. Login successfully 2. Press browser Back button | Admin / admin123 | Stays on dashboard, session maintained | Redirected back to dashboard — session maintained | ✅ Pass |
| TC_018 | Password field masking | 1. Open URL 2. Type in password field | Any text | Characters shown as dots | Password masked correctly | ✅ Pass |
| TC_019 | Narrow browser window (400px) | 1. Open URL 2. Resize browser to 400px wide 3. Try login | Admin / admin123 | Page usable, no overlapping elements | Login form visible and functional | ✅ Pass |
| TC_020 | Login after previous failed attempts | 1. Enter wrong password twice 2. Enter correct credentials | Admin / wrongpass x2, then Admin / admin123 | Login succeeds | Blocked — linked to TC_015 lockout issue | 🚫 Blocked |

---

## Key Observations

1. **No account lockout** — 5 failed attempts did not trigger any lockout or CAPTCHA. Security vulnerability.
2. **Case-sensitive username** — ADMIN fails but Admin works. No message tells the user this.
3. **Whitespace not trimmed** — Leading space in username causes silent login failure.
4. **No frontend length validation** — Username field accepts 300+ characters.
5. **Double-click fires duplicate requests** — Observed directly in Chrome DevTools Network tab.


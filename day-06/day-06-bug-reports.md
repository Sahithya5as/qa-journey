# Day 06 — Bug Reports: Security + Accessibility + Stub Testing
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com

---

## Summary

| Bug ID | Title | Severity | Priority | OWASP / WCAG Ref |
|--------|-------|----------|----------|-----------------|
| BUG-023 | National ID field exposed in employee API response | High | High | OWASP A03 — Sensitive Data Exposure |
| BUG-024 | No session timeout after 30 minutes of inactivity | Medium | Medium | OWASP A02 — Broken Authentication |
| BUG-025 | Error messages not announced by screen reader | Medium | Medium | WCAG 1.3.1 |
| BUG-026 | No fallback when SSO service is unavailable | Medium | High | Reliability |

---

## BUG-023

**Title:** Employee API — National ID field exposed in API response body — sensitive data leakage — Chrome v124 / macOS

**Severity:** High
**Priority:** High
**Type:** Security — Sensitive Data Exposure
**OWASP:** A03 — Sensitive Data Exposure

**Steps to Reproduce:**
1. Login as Admin
2. Open DevTools → Network tab
3. Go to PIM → Employee List
4. Find the API call that fetches employee data
5. Click on it → Response tab
6. Inspect the JSON response body

**Expected Result:**
API response should contain only fields needed by the UI. Sensitive fields like national_id, bank_account, salary should never be returned in list endpoints — only in secure individual profile endpoints with explicit permission checks.

**Actual Result:**
The `national_id` field is visible in the API response body for the employee list endpoint. This data is transmitted to every browser that loads the employee list — regardless of whether the user has permission to view it.

**Impact:**
High — national ID numbers are sensitive PII (Personally Identifiable Information). Exposing them in a list API response violates data privacy regulations including GDPR and Saudi PDPL (Personal Data Protection Law) — directly relevant to Barq operating under Saudi regulations.

---

## BUG-024

**Title:** Authentication — User session does not expire after 30 minutes of inactivity — session timeout not implemented — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** Security — Broken Authentication
**OWASP:** A02 — Broken Authentication

**Steps to Reproduce:**
1. Login as Admin
2. Leave the application completely idle for 30 minutes
3. Attempt to perform any action — navigate to a page, click a button

**Expected Result:**
Session should expire after a defined inactivity timeout (typically 15–30 minutes for HR/financial applications). User should be redirected to login with message: "Your session has expired. Please log in again."

**Actual Result:**
Session remains fully active after 30 minutes of inactivity. User can continue performing actions without re-authenticating.

**Impact:**
Medium — on a shared or public computer, a forgotten logged-in session allows anyone to access sensitive HR data. In a fintech context like Barq, session timeout is a regulatory requirement.

---

## BUG-025

**Title:** Login page — Validation error messages not announced to screen reader users — WCAG 1.3.1 failure — Chrome v124 / macOS

**Severity:** Medium
**Priority:** Medium
**Type:** Accessibility
**WCAG Reference:** Success Criterion 1.3.1 — Info and Relationships (Level A)

**Steps to Reproduce:**
1. Enable VoiceOver (Cmd + F5 on Mac) or NVDA
2. Navigate to login page using keyboard only
3. Submit the form with empty fields
4. Listen for error message announcement

**Expected Result:**
Error message should be announced immediately by the screen reader. The error container should have `role="alert"` or `aria-live="assertive"` so it is announced without the user navigating to it.

**Actual Result:**
Error message appears visually but is not announced by VoiceOver. Screen reader users have no way of knowing the form failed validation unless they manually navigate to the error message.

**Impact:**
Medium — completely blocks visually impaired users from understanding why their login attempt failed. WCAG Level A violation — the minimum required level of accessibility.

---

## BUG-026

**Title:** Login — No fallback mechanism when SSO service is unavailable — blank error page shown — Chrome v124 / macOS

**Severity:** Medium
**Priority:** High
**Type:** Reliability / UX

**Steps to Reproduce:**
1. Simulate SSO service downtime (or disconnect from network briefly)
2. Attempt to login via SSO option

**Expected Result:**
When SSO is unavailable the system should: (a) Fall back to local username/password authentication, OR (b) Show a clear message: "Single sign-on is currently unavailable. Please use your username and password to login."

**Actual Result:**
A generic error page is displayed with no guidance. The user cannot tell if they should retry, use a different login method, or contact support.

**Impact:**
Medium-High — if SSO is the only login method for most users and it goes down, ALL users are locked out with no fallback. In a fintech product this could mean customers cannot access their wallets.


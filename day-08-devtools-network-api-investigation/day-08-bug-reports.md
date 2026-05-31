# Day 08 — Bug Reports: Chrome DevTools Network Investigation
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS | Chrome v148
**Application:** DemoQA — https://demoqa.com
**Found using:** Chrome DevTools Network tab

---

## Summary

| Bug ID | Title | Severity | Priority | OWASP Ref |
|--------|-------|----------|----------|-----------|
| BUG-D001 | Password returned in plain text in Login API response | Critical | P1 | A02, A03 |
| BUG-D002 | Password embedded inside JWT token payload | Critical | P1 | A02 |
| BUG-D003 | JWT token has no expiry claim — valid forever | High | P1 | A02 |
| BUG-D004 | Inactive user can generate valid auth token | High | P2 | A02 |
| BUG-D005 | Invalid credentials return 200 OK instead of 401 | High | P2 | A02 |
| BUG-D006 | Form submission fires no network request — data not persisted | Medium | P3 | — |
| BUG-D007 | Book description truncated mid-sentence in API response | Low | P4 | — |

---

## BUG-D001 — CRITICAL

**Title:** Login API — User password returned in plain text in `/Account/v1/Login` response body — OWASP A03 Sensitive Data Exposure — Chrome v148 / macOS

**Severity:** Critical
**Priority:** P1
**Type:** Security — Sensitive Data Exposure
**OWASP:** A02 Broken Authentication + A03 Sensitive Data Exposure
**Found by:** Chrome DevTools Network tab → Response tab investigation

**Steps to Reproduce:**
1. Open Chrome → Navigate to https://demoqa.com/login
2. Open DevTools → Network tab → Fetch/XHR filter
3. Enter valid credentials and click Login
4. Click the POST request to `/Account/v1/Login`
5. Click the Response tab

**Expected Result:**
Response should contain only token, expiry, userId, and status. Password should never appear in any API response under any circumstances.

**Actual Result:**
```json
{
    "userId": "4484b0ca-b448-4b99-a255-a6a2cee69039",
    "username": "testqa2026",
    "password": "Test@12345",
    "token": "eyJhbGci...",
    "expires": "2026-06-06T16:58:40.000Z"
}
```
The user's actual plain text password is returned in the response body.

**Impact:**
Critical — any network log, browser history, proxy tool, or man-in-the-middle attack captures this response and immediately has the user's password. In a financial application this is a catastrophic security failure.

---

## BUG-D002 — CRITICAL

**Title:** Authentication — User password embedded inside JWT token payload — token is decodable by anyone — Chrome v148 / macOS

**Severity:** Critical
**Priority:** P1
**Type:** Security — Broken Authentication
**OWASP:** A02 Broken Authentication
**Found by:** jwt.io token decoder used on token captured in DevTools

**Steps to Reproduce:**
1. Login and capture the token from the GenerateToken response
2. Go to https://jwt.io
3. Paste the full token in the decoder
4. Observe the Payload section

**Expected Result:**
JWT payload should contain only userId, role, and expiry. Never sensitive credentials.

**Actual Result:**
```json
{
  "userName": "testqa2026",
  "password": "Test@12345",
  "iat": 1780160320
}
```
Full plain text password visible in the decoded JWT payload.

**Impact:**
Critical — JWT tokens are base64 encoded, NOT encrypted. Any person who obtains the token (from logs, network capture, shared URL) can instantly decode it and retrieve the user's password. The password is then usable on any other service where the user reuses the same credentials.

---

## BUG-D003 — HIGH

**Title:** Authentication — JWT token missing `exp` (expiry) claim — tokens are valid indefinitely — Chrome v148 / macOS

**Severity:** High
**Priority:** P1
**Type:** Security — Broken Authentication
**OWASP:** A02 Broken Authentication
**Found by:** jwt.io decoder — checking JWT claims

**Steps to Reproduce:**
1. Login and capture the JWT token
2. Decode at jwt.io
3. Check the Payload section for `exp` claim

**Expected Result:**
JWT should contain an `exp` claim defining when the token expires — typically 15 minutes to 24 hours depending on the application's security policy.

**Actual Result:**
JWT payload contains only `userName`, `password`, and `iat` (issued at). No `exp` claim present. Token never expires.

**Impact:**
High — if a token is stolen, the attacker has permanent access to the account with no expiry. Standard security practice requires short-lived tokens (15–60 minutes) with refresh token mechanisms. Without expiry, token theft is permanently damaging.

---

## BUG-D004 — HIGH

**Title:** Authentication — User with `isActive: false` flag can successfully generate auth token and access the application — Chrome v148 / macOS

**Severity:** High
**Priority:** P2
**Type:** Security — Broken Authentication
**OWASP:** A02 Broken Authentication
**Found by:** Observing isActive field in Login API response

**Steps to Reproduce:**
1. Login with valid credentials
2. Check the `/Account/v1/Login` response body
3. Observe `"isActive": false`
4. Confirm the user is still fully authenticated and token is valid

**Expected Result:**
Users with `isActive: false` should be blocked from authentication. Response should be 403 Forbidden with message: "Account is inactive. Please contact your administrator."

**Actual Result:**
Valid token generated and full application access granted despite `isActive: false`. The isActive flag has no effect on authentication.

**Impact:**
High — deactivated accounts (former employees, suspended users, compromised accounts) should immediately lose access. If the isActive flag is used for account suspension, it is currently completely ineffective.

---

## BUG-D005 — HIGH

**Title:** Authentication — Invalid credentials return HTTP 200 OK instead of 401 Unauthorized — API contract violation — Chrome v148 / macOS

**Severity:** High
**Priority:** P2
**Type:** API Contract Violation
**OWASP:** A02 Broken Authentication
**Found by:** Chrome DevTools Network tab — Status column

**Steps to Reproduce:**
1. Go to https://demoqa.com/login
2. Enter wrong credentials: username `wrong` password `wrong`
3. Click Login
4. Check the Status column in the Network tab for the GenerateToken request

**Expected Result:**
HTTP 401 Unauthorized — the standard HTTP status code for failed authentication.

**Actual Result:**
HTTP 200 OK with response body:
```json
{
    "token": null,
    "expires": null,
    "status": "Failed",
    "result": "User authorization failed."
}
```

**Impact:**
High — any client application (mobile app, third-party integration) that checks the HTTP status code to determine login success would incorrectly treat this as a successful response. The error is hidden in the response body, not surfaced at the protocol level. This is a standard REST API contract violation.

---

## BUG-D006 — MEDIUM

**Title:** Automation Practice Form — Form submission fires no network request — entered data is never sent to or saved by the server — Chrome v148 / macOS

**Severity:** Medium
**Priority:** P3
**Type:** Functional — No backend integration
**Found by:** Chrome DevTools Network tab — observing zero XHR requests on form submit

**Steps to Reproduce:**
1. Go to https://demoqa.com/automation-practice-form
2. Open DevTools → Network tab → Fetch/XHR filter
3. Fill in all form fields completely
4. Click Submit
5. Observe the Network tab

**Expected Result:**
A POST request fires to a server endpoint with the form data in the payload. A success response confirms the data was saved.

**Actual Result:**
Zero network requests fire. A modal popup appears confirming submission but no data is ever sent to any server. The form is purely client-side with no backend integration.

**Impact:**
Medium — misleads users into thinking their data was saved when it was not. Any automation test validating form data persistence would produce a false positive. Testers must always verify data persistence via API or database — never rely on UI confirmation alone.

---

## BUG-D007 — LOW

**Title:** BookStore API — Book description field truncated mid-sentence at approximately 200 characters in API response — Chrome v148 / macOS

**Severity:** Low
**Priority:** P4
**Type:** Data Integrity
**Found by:** Chrome DevTools → Response tab inspection of /BookStore/v1/Books

**Steps to Reproduce:**
1. Go to https://demoqa.com/books
2. Open DevTools → Network tab → Fetch/XHR filter
3. Find GET /BookStore/v1/Books request
4. Click Response tab
5. Observe the description field of any book

**Expected Result:**
Full book description returned without truncation.

**Actual Result:**
Description for Git Pocket Guide ends mid-sentence:
`"...those of you with Git exp"`
Full description cut off at approximately 200 characters.

**Impact:**
Low — book descriptions incomplete in the API response. Any application displaying these descriptions would show incomplete content to users. May affect SEO if descriptions are used in page content.


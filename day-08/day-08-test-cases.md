# Day 08 — Test Cases: Chrome DevTools Network Investigation
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS | Chrome v148
**Application:** DemoQA — https://demoqa.com
**Method:** Chrome DevTools Network tab investigation

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 22 | 13 | 7 | 2 |

---

## Section 1 — Authentication API Tests

| TC ID | Scenario | Endpoint | Method | Expected | Actual | Status |
|-------|----------|----------|--------|----------|--------|--------|
| TC_N_001 | Valid login generates token | POST /Account/v1/GenerateToken | POST | 200 OK, token returned, status: Success | 200 OK, token returned | ✅ Pass |
| TC_N_002 | Invalid credentials returns 401 | POST /Account/v1/GenerateToken | POST | 401 Unauthorized | 200 OK with status: Failed, token: null | ⚠️ Fail |
| TC_N_003 | Login response does not expose password | POST /Account/v1/Login | POST | Password not in response body | Password visible in plain text in response | ⚠️ Fail |
| TC_N_004 | JWT token does not contain password | POST /Account/v1/GenerateToken | POST | JWT payload contains only userId and expiry | JWT payload contains username AND password | ⚠️ Fail |
| TC_N_005 | JWT token has expiry claim | POST /Account/v1/GenerateToken | POST | Token contains exp claim | No exp claim — token never expires | ⚠️ Fail |
| TC_N_006 | Inactive user cannot authenticate | POST /Account/v1/GenerateToken | POST | 401 or 403 for inactive user | Token generated — isActive: false ignored | ⚠️ Fail |
| TC_N_007 | Login response time under 500ms | POST /Account/v1/GenerateToken | POST | < 500ms | 289ms | ✅ Pass |

---

## Section 2 — Book List API Tests

| TC ID | Scenario | Endpoint | Method | Expected | Actual | Status |
|-------|----------|----------|--------|----------|--------|--------|
| TC_N_008 | Book list loads without authentication | GET /BookStore/v1/Books | GET | 200 OK, books returned | 200 OK, 8 books returned | ✅ Pass |
| TC_N_009 | Book list returns correct fields | GET /BookStore/v1/Books | GET | isbn, title, author, publisher, pages, description, website | All fields present | ✅ Pass |
| TC_N_010 | Description field not truncated | GET /BookStore/v1/Books | GET | Full description returned | Description cut off mid-sentence ~200 chars | ⚠️ Fail |
| TC_N_011 | Book list response time under 500ms | GET /BookStore/v1/Books | GET | < 500ms | 189ms | ✅ Pass |
| TC_N_012 | Book list returns 8 books | GET /BookStore/v1/Books | GET | All books in catalogue returned | 8 books returned | ✅ Pass |

---

## Section 3 — Protected API Tests

| TC ID | Scenario | Endpoint | Method | Expected | Actual | Status |
|-------|----------|----------|--------|----------|--------|--------|
| TC_N_013 | Profile requires Authorization header | GET /Account/v1/User/{id} | GET | Request includes Bearer token | Bearer token sent in Authorization header | ✅ Pass |
| TC_N_014 | Profile does not expose password | GET /Account/v1/User/{id} | GET | Password not in response | Only userId, username, books returned | ✅ Pass |
| TC_N_015 | Profile response time under 500ms | GET /Account/v1/User/{id} | GET | < 500ms | 295ms | ✅ Pass |
| TC_N_016 | Sensitive data not in URL | GET /Account/v1/User/{id} | GET | No token or password in URL | URL contains only userId — no sensitive data | ✅ Pass |

---

## Section 4 — Form and Upload Tests

| TC ID | Scenario | Page | Expected | Actual | Status |
|-------|----------|------|----------|--------|--------|
| TC_N_017 | Empty form triggers validation before API call | /automation-practice-form | Validation shown, no network request | No network request fired — correct | ✅ Pass |
| TC_N_018 | Valid form submission sends POST request | /automation-practice-form | POST request fires with form data | No network request fires at all | ⚠️ Fail |
| TC_N_019 | File upload sends multipart request | /upload-download | POST with multipart/form-data | No network request fires | 🚫 Blocked |

---

## Section 5 — Performance and Security Tests

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_N_020 | All API calls under 300ms | < 300ms | Max 295ms | ✅ Pass |
| TC_N_021 | No duplicate requests on double-click login | 1 POST request | 1 POST request fired | ✅ Pass |
| TC_N_022 | Auth token not visible in browser URL | No token in URL | URL shows /profile only | ✅ Pass |


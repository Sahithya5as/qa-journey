# Day 08 — Chrome DevTools Network Tab — Real Investigation
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS | Chrome v148
**Application Tested:** DemoQA — https://demoqa.com
**OrangeHRM was unavailable — DemoQA used as alternative**

---

## What is the Chrome DevTools Network Tab

Every time you interact with a web application, the browser makes API calls to the server behind the scenes. The UI shows you the polished result. The Network tab shows you everything underneath — the raw requests, the responses, the status codes, the headers, and the payloads.

This is the most powerful tool a manual QA engineer has. It lets you:
- See what data the browser is actually sending
- See what the server is actually returning
- Find bugs the UI completely hides from you
- Validate API responses against documented contracts
- Catch security issues before they reach production

**How to open:**
Press `Cmd + Option + I` → Network tab → tick Preserve Log → tick Disable Cache

---

## Network Tab Anatomy

### Columns in the request list

| Column | What it shows |
|--------|--------------|
| Name | API endpoint URL |
| Method | GET / POST / PUT / PATCH / DELETE |
| Status | HTTP response code |
| Type | xhr/fetch = API call, document = page, script/css = assets |
| Size | Data transferred |
| Time | Total request duration |

### Tabs inside each request

| Tab | What it shows |
|-----|--------------|
| Headers | Request + response headers including auth token |
| Payload | Data SENT in the request body |
| Response | Raw data returned by the server |
| Preview | Pretty-printed response |
| Timing | Breakdown of where time was spent |

### Filter tip
Click **Fetch/XHR** to show only API calls — hides all CSS, JS, images, and ad noise.
Type a domain in the filter box (e.g. `demoqa.com`) to hide third-party requests.

---

## HTTP Status Codes — Must Know Cold

| Code | Meaning | Whose fault |
|------|---------|-------------|
| 200 | OK — success | Nobody — correct |
| 201 | Created — new resource created | Nobody — correct |
| 204 | No content — success, no body | Nobody — correct |
| 400 | Bad Request — wrong data sent | Client / frontend |
| 401 | Unauthorized — not authenticated | Client needs to login |
| 403 | Forbidden — authenticated but no permission | Server — access control |
| 404 | Not Found | Could be either |
| 422 | Unprocessable Entity — validation failed | Client sent invalid data |
| 500 | Internal Server Error — server crashed | Backend |
| 503 | Service Unavailable | Infrastructure |

---

## What I Investigated Today — DemoQA Bookstore App

### Application Overview
DemoQA is a QA practice application with a Bookstore module. It has:
- User registration and authentication
- Book catalogue (public)
- User profile with personalised book collection (protected)
- Practice forms and UI components

---

## Task 1 — Login API Investigation

### Endpoints discovered
```
POST /Account/v1/GenerateToken  — generates auth token
POST /Account/v1/Login          — authenticates and returns user object
```

### GenerateToken Response (successful login)
```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires": "2026-06-06T16:58:40.256Z",
    "status": "Success",
    "result": "User authorized successfully."
}
```

### Login Response (successful login)
```json
{
    "userId": "4484b0ca-b448-4b99-a255-a6a2cee69039",
    "username": "testqa2026",
    "password": "Test@12345",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires": "2026-06-06T16:58:40.000Z",
    "created_date": "2026-05-30T16:54:35.000Z",
    "isActive": false
}
```

### Critical Finding — Password in plain text
The `/Account/v1/Login` response returns the user's actual password in plain text in the response body. This was confirmed by seeing `"password": "Test@12345"` in the Response tab.

### JWT Token Analysis
Decoded the token at jwt.io. The JWT payload contains:
```json
{
  "userName": "testqa2026",
  "password": "Test@12345",
  "iat": 1780160320
}
```

**Key observations:**
- Password is embedded inside the JWT token
- JWT is base64 encoded — NOT encrypted — anyone can decode it instantly
- No `exp` (expiry) claim present — token never expires
- `iat` (issued at) present — only records creation time

### Invalid credentials behaviour
When wrong credentials are submitted:
- Status code returned: **200 OK** (incorrect)
- Expected: **401 Unauthorized**
- Response body shows `"status": "Failed"` and `"token": null`
- This is an API contract violation — HTTP status code must reflect the outcome

### isActive: false
User account shows `"isActive": false` yet the system still generated a valid token and allowed authentication. Inactive users should not be able to authenticate.

### Response time: 289ms ✅

---

## Task 2 — Book List API Investigation

### Endpoint
```
GET /BookStore/v1/Books
```

### Response structure
```json
{
  "books": [
    {
      "isbn": "9781449325862",
      "title": "Git Pocket Guide",
      "subTitle": "A Working Introduction",
      "author": "Richard E. Silverman",
      "publish_date": "2020-06-04T08:48:39.000Z",
      "publisher": "O'Reilly Media",
      "pages": 234,
      "description": "This pocket guide is the perfect on-the-job...",
      "website": "..."
    }
  ]
}
```

**8 books returned. No Authorization header required.**

### Public vs Protected endpoints
This endpoint is intentionally public — no auth required to browse the book catalogue. This is correct design — equivalent to browsing an e-commerce store without logging in. Protected endpoints (profile, add book) correctly require Bearer token authentication.

### Finding — Description truncated
The description field is cut off mid-sentence at approximately 200 characters. Full description not returned.

### Response time: 189ms ✅

---

## Task 3 — Form Submission Investigation

### Automation Practice Form
Tested at `/automation-practice-form`. Both empty and valid form submissions were observed in the Network tab. **No network requests fired on form submission.** The form shows a confirmation modal but never sends data to any server endpoint. The form is client-side only with no backend integration.

**QA implication:** Any automation test asserting that form data was saved would produce a false positive. Always verify data persistence via an API call or database check — not just UI confirmation.

### File Upload
Tested at `/upload-download`. File selection triggered no network request. The upload functionality is not connected to a backend endpoint.

---

## Task 4 — Security Investigation

### Authorization header on protected endpoints
```
GET /Account/v1/User/{userId}
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Protected endpoints correctly require Bearer token in Authorization header. The token from login is automatically passed in subsequent requests.

### User profile response
```json
{
    "userId": "4484b0ca-b448-4b99-a255-a6a2cee69039",
    "username": "testqa2026",
    "books": []
}
```

Password is NOT returned in the User profile endpoint — only in the Login endpoint. This shows inconsistent security behaviour across endpoints.

### URL security
Profile URL shows `/profile` only — no sensitive data (token, userId, password) in the browser URL. Correct behaviour.

---

## Task 5 — Performance Summary

| Endpoint | Method | Response Time | TTFB | Assessment |
|----------|--------|--------------|------|------------|
| /Account/v1/GenerateToken | POST | 289ms | ~250ms | ✅ Acceptable |
| /BookStore/v1/Books | GET | 189ms | ~150ms | ✅ Fast |
| /Account/v1/User/{id} | GET | 295ms | ~260ms | ✅ Acceptable |

All endpoints under 300ms — well within the 2-second threshold for API responses. High TTFB relative to total time suggests server processing time dominates over payload download time — payload sizes are small.

---

## Task 6 — Duplicate Request Test

Double-clicking the Login button fired only ONE POST request to `/Account/v1/GenerateToken`. The button is correctly protected against duplicate submission. No duplicate request bug found.

---

## Key Concepts Learned Today

### Public vs Protected endpoints
Not all APIs require authentication. Book lists, product catalogues, and public content are intentionally unauthenticated. User-specific data (profile, orders, transactions) must always require authentication.

### JWT Token structure
A JWT has three parts separated by dots:
```
header.payload.signature
eyJhbGci... . eyJ1c2VyT... . OLhcnDk_...
```
- Header: algorithm used
- Payload: the data (claims) — base64 encoded, NOT encrypted
- Signature: verifies the token has not been tampered with

**Important:** Anyone can decode a JWT payload. Never put sensitive data inside it.

### TTFB vs Content Download
- High TTFB (Waiting) = server is slow to process
- High Content Download = response payload is too large
- Today's TTFB was high relative to download — server processing dominates

### Ad noise in Network tab
Third-party ad networks (doubleclick.net etc.) flood the Network tab. Always filter by domain (`demoqa.com`) or use Fetch/XHR filter to isolate API calls from noise.

---

## Interview Talking Points

- Can open DevTools Network tab, filter to API calls only, and investigate any request end to end
- Know all HTTP status codes and can identify incorrect ones (200 on failed auth)
- Can decode a JWT token and identify security issues in its payload
- Know the difference between public and protected API endpoints
- Understand TTFB vs Content Download in performance analysis
- Found 7 real bugs including 2 Critical security vulnerabilities from a single session
- Know how to filter ad noise from Network tab to find relevant API calls


# Day 06 — Test Cases: Non-Functional, Security, BDD Scenarios
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** Non-functional testing, OWASP security, BDD Gherkin scenarios

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 20 | 14 | 4 | 2 |

---

## Section 1 — Performance Observation Tests

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_PF_001 | Performance | Login response time — normal network | Login, measure time in DevTools | < 2 seconds | 1.2 seconds | ✅ Pass |
| TC_PF_002 | Performance | Login — 3G throttled network | Throttle to 3G in DevTools, login | < 5 seconds | 4.2 seconds — acceptable | ✅ Pass |
| TC_PF_003 | Performance | Employee list load — 100 records | Go to PIM → Employee List, measure load | < 3 seconds | 2.8 seconds | ✅ Pass |
| TC_PF_004 | Performance | Leave list load — large dataset | Go to Leave → Leave List, measure | < 3 seconds | 3.4 seconds — slightly over threshold | ⚠️ Note |

---

## Section 2 — OWASP Security Tests

| TC ID | Type | OWASP | Scenario | Steps | Expected | Actual | Status |
|-------|------|-------|----------|-------|----------|--------|--------|
| TC_SEC_001 | Security | A03 — Injection | SQL injection in username | Enter ' OR '1'='1 in username | Normal error — no DB error exposed | Handled correctly — no DB error | ✅ Pass |
| TC_SEC_002 | Security | A07 — XSS | Script injection in comment field | Enter `<script>alert(1)</script>` in leave comment | Script not executed — rendered as text | Rendered as text — not executed | ✅ Pass |
| TC_SEC_003 | Security | A01 — Broken Access Control | Horizontal privilege escalation | Login as ESS user, navigate to /admin/viewAdminModule | Access blocked | Blocked — redirected to dashboard | ✅ Pass |
| TC_SEC_004 | Security | A02 — Broken Auth | Auth token visibility | Login, check DevTools → Network → Response headers | Token in header only — not in response body | Token in header only — correct | ✅ Pass |
| TC_SEC_005 | Security | A05 — Security Misconfiguration | Verbose error messages | Enter invalid data, observe error | Generic error — no internal path or stack trace exposed | Generic error shown correctly | ✅ Pass |
| TC_SEC_006 | Security | A03 — Sensitive data | PII in API response | Call employee API, check response body | No salary, bank account, or national ID in response | National ID field visible in response — not expected | ⚠️ Fail |
| TC_SEC_007 | Security | A02 — Session | Session expiry after inactivity | Leave app idle 30 minutes, attempt action | Redirected to login | Session remained active — no timeout | ⚠️ Fail |
| TC_SEC_008 | Security | A02 — Auth | Account lockout after failed attempts | 5 wrong passwords | Account locked or CAPTCHA | No lockout — linked to BUG-001 still open | ⚠️ Blocked |

---

## Section 3 — Accessibility Tests (WCAG 2.1)

| TC ID | Type | WCAG Ref | Scenario | Expected | Actual | Status |
|-------|------|----------|----------|----------|--------|--------|
| TC_AC_001 | Accessibility | 2.1.1 | Keyboard navigation — full login flow | Tab → fields, Enter → submit | Works correctly | ✅ Pass |
| TC_AC_002 | Accessibility | 1.3.1 | Error messages announced by screen reader | Error has aria-live or role=alert | Error not announced | ⚠️ Fail |
| TC_AC_003 | Accessibility | 1.3.1 | Form fields have labels | Each input has associated label | All inputs labelled | ✅ Pass |
| TC_AC_004 | Accessibility | 2.4.3 | Focus visible on keyboard navigation | Visible focus outline on active element | Focus visible — blue outline shown | ✅ Pass |

---

## Section 4 — BDD Gherkin Scenarios
*Written to demonstrate BDD knowledge — these would be implemented with Behave/Cucumber*

```gherkin
Feature: Leave Application

  Scenario: Employee applies for leave with sufficient balance
    Given the employee "EMP001" has 5 days of annual leave remaining
    When the employee applies for 2 days of leave from next Monday to Tuesday
    Then the leave request should be created with status "Pending"
    And the employee's leave balance should remain at 5 days until approved

  Scenario: Manager approves a valid leave request
    Given a leave request exists with status "Pending" for employee "EMP001"
    And the employee has sufficient leave balance
    When the manager approves the request
    Then the leave status should change to "Approved"
    And the employee's leave balance should be reduced by 2 days
    And an email notification should be sent to the employee

  Scenario: Manager cannot approve leave when employee has zero balance
    Given employee "EMP002" has 0 days of annual leave remaining
    And a leave request exists for employee "EMP002"
    When the manager attempts to approve the request
    Then the system should display "Insufficient leave balance"
    And the leave status should remain "Pending"

  Scenario: Employee cannot apply leave on public holidays
    Given a public holiday exists on the selected leave dates
    When the employee applies for leave covering those dates
    Then the system should warn "Your dates include public holidays"
    And the public holiday dates should not be counted against the leave balance
```

---

## Section 5 — Stub and Mock Scenarios
*Testing what happens when dependencies are unavailable — stub/mock behaviour*

| TC ID | Type | Scenario | Dependency Stubbed | Expected | Actual | Status |
|-------|------|----------|-------------------|----------|--------|--------|
| TC_ST_001 | Stub | Leave approval when email service down | Email service stubbed — returns timeout | Leave approved successfully, email failure logged | Leave approved — email error logged silently | ✅ Pass |
| TC_ST_002 | Stub | Login when SSO service unavailable | SSO service stubbed — returns 503 | Fallback to local auth shown to user | Error page shown — no fallback | ⚠️ Fail |


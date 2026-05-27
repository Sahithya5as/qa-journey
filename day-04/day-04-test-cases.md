# Day 04 — Test Cases: Test Planning, Risk-Based Testing, Entry/Exit Criteria
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** Risk-based test prioritisation, shift-left findings, entry/exit criteria validation

---

## Risk Assessment Before Testing

| Module | Probability of Failure | Business Impact | Risk Level | Test Priority |
|--------|----------------------|-----------------|------------|---------------|
| Leave approval + balance | High — bugs found previously | High — payroll impact | 🔴 High | Full coverage |
| Authentication | Medium — security bugs found | High — access control | 🔴 High | Full coverage |
| Recruitment module | Low — unchanged this sprint | Medium | 🟡 Medium | Core scenarios only |
| Dashboard widgets | Low | Low | 🟢 Low | Smoke test only |

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 18 | 13 | 3 | 2 |

---

## Section 1 — High Risk: Leave Approval (Full Coverage)

| TC ID | Risk | Scenario | Steps | Test Data | Expected | Actual | Status |
|-------|------|----------|-------|-----------|----------|--------|--------|
| TC_071 | 🔴 | Approve leave with sufficient balance | 1. Set balance = 5 days 2. Apply 2 days 3. Approve | Balance: 5, Leave: 2 days | Approved, balance = 3 | Balance correctly updated to 3 | ✅ Pass |
| TC_072 | 🔴 | Approve leave at exact balance boundary | 1. Set balance = 3 days 2. Apply 3 days 3. Approve | Balance: 3, Leave: 3 days | Approved, balance = 0 | Balance shows 0 correctly | ✅ Pass |
| TC_073 | 🔴 | Approve leave with zero balance | 1. Set balance = 0 2. Apply 2 days 3. Approve | Balance: 0, Leave: 2 days | Warning shown or blocked | Approved with no warning — balance shows -2 | ⚠️ Fail |
| TC_074 | 🔴 | Overlapping leave requests same dates | 1. Apply Mon–Wed 2. Without approving apply same dates again | Same employee same dates | Second request rejected | Both requests accepted as Pending | ⚠️ Fail |
| TC_075 | 🔴 | Leave approval email notification sent | 1. Apply leave 2. Approve 3. Check employee email | Valid leave request | Email notification sent to employee | Email sent to employee | ✅ Pass |
| TC_076 | 🔴 | Leave rejection with reason saved | 1. Apply leave 2. Reject with reason: "Team on leave" | Valid leave request | Reason saved and visible to employee | Reason saved and shown correctly | ✅ Pass |

---

## Section 2 — High Risk: Authentication (Full Coverage)

| TC ID | Risk | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_077 | 🔴 | Login after password reset | 1. Reset via Forgot Password 2. Login with new password | Login succeeds | Login succeeded with new password | ✅ Pass |
| TC_078 | 🔴 | Concurrent sessions same user | 1. Login in Chrome 2. Login in Firefox with same credentials | Second login invalidates first OR warning shown | Both sessions active simultaneously | ⚠️ Fail |
| TC_079 | 🔴 | HTTPS enforced on login | 1. Navigate to http:// version of URL | Redirected to https:// | Redirected to HTTPS correctly | ✅ Pass |
| TC_080 | 🔴 | Password not in page source | 1. Login 2. View Page Source | No credentials in source code | No credentials in source | ✅ Pass |
| TC_081 | 🔴 | Direct URL without login blocked | 1. Without logging in navigate to /pim/viewEmployeeList | Redirected to login page | Correctly redirected to login | ✅ Pass |

---

## Section 3 — Medium Risk: Recruitment (Core Scenarios)

| TC ID | Risk | Scenario | Expected | Actual | Status |
|-------|------|----------|----------|--------|--------|
| TC_082 | 🟡 | Create job vacancy | Vacancy created successfully | Created | ✅ Pass |
| TC_083 | 🟡 | Add candidate record | Candidate record created | Created | ✅ Pass |
| TC_084 | 🟡 | Schedule interview | Interview scheduled with date and interviewer | Scheduled correctly | ✅ Pass |
| TC_085 | 🟡 | Move candidate through hiring stages | Status updates at each stage | All stages working | ✅ Pass |

---

## Section 4 — Low Risk: Dashboard (Smoke Only)

| TC ID | Risk | Scenario | Expected | Actual | Status |
|-------|------|----------|----------|--------|--------|
| TC_086 | 🟢 | Dashboard loads after login | Widgets visible, no errors | Loaded correctly | ✅ Pass |
| TC_087 | 🟢 | Quick launch shortcuts work | Each icon navigates to correct module | All icons working | ✅ Pass |

---

## Section 5 — Shift-Left Findings (Requirement Gaps Found Before Coding)

These gaps were identified during requirement review — before any development started. This is shift-left QA in practice.

| ID | Type | Gap Found | Impact if Not Fixed |
|----|------|-----------|---------------------|
| SL_001 | Missing requirement | No AC defined for what happens when employee has no leave type assigned | Developer implements undefined behaviour — likely a crash |
| SL_002 | Security gap | Password reset token expiry not specified | Security risk — reset links valid indefinitely |
| SL_003 | Not testable | "System should respond quickly" — no performance threshold defined | Cannot write pass/fail test case for this requirement |
| SL_004 | Missing requirement | No requirement for what happens to leave requests when employee is deleted | Orphaned data in production |
| SL_005 | Ambiguous AC | "Managers can approve leave" — does not specify which manager (direct only or any?) | Incorrect access control implementation |

---

## Exit Criteria Assessment — End of Day 04

| Criterion | Target | Actual | Met? |
|-----------|--------|--------|------|
| Test execution rate | ≥ 95% | 100% (18/18) | ✅ |
| Critical bugs open | 0 | 0 | ✅ |
| High bugs open | 0 | 2 open | ❌ |
| Regression passing | 100% | 94% | ⚠️ |
| Test summary ready | Yes | Yes | ✅ |

**Release Recommendation:** Do not release. TC_073 (zero balance approved) and TC_078 (concurrent sessions) must be investigated and fixed before release.


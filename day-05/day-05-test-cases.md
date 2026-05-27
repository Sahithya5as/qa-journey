# Day 05 — Test Cases: Testing Types Applied on OrangeHRM
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** Demonstrating smoke, sanity, regression, retest, exploratory, gorilla testing types

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 22 | 15 | 5 | 2 |

---

## Section 1 — Smoke Test Suite
*Run first on every new build. If any fail — reject build immediately, no further testing.*

| TC ID | Type | Scenario | Steps | Expected | Actual | Status |
|-------|------|----------|-------|----------|--------|--------|
| TC_SM_001 | Smoke | Application accessible | Open URL | Login page loads | Loaded | ✅ Pass |
| TC_SM_002 | Smoke | Admin login works | Login Admin/admin123 | Dashboard loads | Dashboard loaded | ✅ Pass |
| TC_SM_003 | Smoke | PIM module accessible | Go to PIM → Employee List | Employee list loads | Loaded | ✅ Pass |
| TC_SM_004 | Smoke | Leave module accessible | Go to Leave → Apply Leave | Form loads | Loaded | ✅ Pass |
| TC_SM_005 | Smoke | Create employee — core flow | PIM → Add Employee → fill name → Save | Employee created | Created successfully | ✅ Pass |
| TC_SM_006 | Smoke | Logout works | Click Logout | Redirected to login page | Redirected | ✅ Pass |

**Smoke Result:** ✅ PASS — Build stable. Proceed to full testing.

---

## Section 2 — Sanity Test Suite
*Run after BUG-007 fix — verify the fix works, check immediate neighbours*

| TC ID | Type | Bug Fixed | Steps | Expected | Actual | Status |
|-------|------|-----------|-------|----------|--------|--------|
| TC_SN_001 | Sanity | BUG-007 | Set balance = 0, apply leave, try to approve | Warning shown, approval blocked | Warning shown — fix verified | ✅ Pass |
| TC_SN_002 | Sanity | BUG-007 | Set balance = 5, apply 2 days, approve | Approved successfully | Approved correctly | ✅ Pass |
| TC_SN_003 | Sanity | BUG-007 | Approve 2-day leave from 5-day balance, check balance | Balance shows 3 | Balance correctly shows 3 | ✅ Pass |

**Sanity Result:** ✅ PASS — BUG-007 fix verified. Proceed to regression.

---

## Section 3 — Regression Test Suite
*Run before every release — verify nothing broke after recent changes*

| TC ID | Type | Module | Scenario | Expected | Actual | Status |
|-------|------|--------|----------|----------|--------|--------|
| TC_RG_001 | Regression | Login | Valid login | Dashboard loads | ✅ | ✅ Pass |
| TC_RG_002 | Regression | Login | Invalid credentials | Error shown | ✅ | ✅ Pass |
| TC_RG_003 | Regression | Login | SQL injection attempt | Normal error — no DB exposure | ✅ | ✅ Pass |
| TC_RG_004 | Regression | PIM | Create employee | Employee created | ✅ | ✅ Pass |
| TC_RG_005 | Regression | PIM | Search employee by name | Correct results returned | ✅ | ✅ Pass |
| TC_RG_006 | Regression | Leave | Apply leave | Request created as Pending | ✅ | ✅ Pass |
| TC_RG_007 | Regression | Leave | Approve with balance | Approved, balance updated | ✅ | ✅ Pass |
| TC_RG_008 | Regression | Leave | BUG-007 regression | Zero balance blocks approval | ✅ | ✅ Pass |

**Regression Result:** ✅ 8/8 PASS — No regression detected.

---

## Section 4 — Retest Suite
*Exact re-execution of specific bug test cases after developer marks Fixed*

| TC ID | Type | Bug | Original Issue | Retest Steps | Expected | Actual | Status |
|-------|------|-----|----------------|--------------|----------|--------|--------|
| TC_RT_001 | Retest | BUG-005 | Double-click fired 2 POST requests | Double-click Login → check DevTools Network tab | Only 1 POST request fires | 2 requests still firing | ⚠️ Reopen |
| TC_RT_002 | Retest | BUG-006 | Vague "Invalid" date error | Set end date before start → submit | Specific error message shown | Still shows generic "Invalid" | ⚠️ Reopen |
| TC_RT_003 | Retest | BUG-008 | No feedback on cancelled leave approval | Approve a cancelled leave request | Clear error message shown | Clear error shown — FIXED | ✅ Closed |

---

## Section 5 — Exploratory Testing Session
*Charter: Leave cancellation edge cases — 60 minute time-box*

**Charter written before session:**
```
Goal: Explore leave cancellation and edge case flows
Scope: Apply leave → cancel → reapply, public holidays, zero balance scenarios
Time-box: 60 minutes
Heuristic: SFDIPOT — focus on Data (D) and Integration (I) today
```

**Findings during session:**

| TC ID | Type | Finding | Severity | Raised As |
|-------|------|---------|----------|-----------|
| TC_EX_001 | Exploratory | Leave applied on public holiday — counted as working day | Medium | BUG-017 |
| TC_EX_002 | Exploratory | Leave type with zero entitlement still selectable in dropdown | Low | BUG-018 |
| TC_EX_003 | Exploratory | Page refresh mid-application loses all entered data — no warning | Medium | BUG-019 |
| TC_EX_004 | Exploratory | Employee comments on leave application not visible to manager | Medium | BUG-020 |

**Session Summary:**
Duration: 60 minutes. Bugs found: 4. All 4 were NOT covered by any existing scripted test case.
This demonstrates why exploratory testing runs alongside scripted testing — it finds what scripts miss.

---

## Section 6 — Gorilla Testing: Login Module Exhaustive Coverage

| TC ID | Type | Scenario | Expected | Actual | Status |
|-------|------|----------|----------|--------|--------|
| TC_GO_001 | Gorilla | Login with keyboard only — no mouse | Tab to fields, Enter submits | Works correctly | ✅ Pass |
| TC_GO_002 | Gorilla | Login with browser autofill | Correct fields filled | Username and password filled correctly | ✅ Pass |
| TC_GO_003 | Gorilla | Login with password manager | Password manager detects field | Detected and populated | ✅ Pass |
| TC_GO_004 | Gorilla | Login after clearing browser cache | Fresh login — no cached session | Login page shown correctly | ✅ Pass |
| TC_GO_005 | Gorilla | Login on 3G throttled network | Works — response under 5s | 4.2 seconds — acceptable | ✅ Pass |
| TC_GO_006 | Gorilla | Login with JavaScript disabled | Helpful message or redirect | Blank white page — no message | ⚠️ Fail |
| TC_GO_007 | Gorilla | Login at 200% browser zoom | All elements usable | Elements overlap — form breaks | ⚠️ Fail |
| TC_GO_008 | Gorilla | Login on 375px mobile screen | Form usable on small screen | Usable but CTA button very small | ⚠️ Note |


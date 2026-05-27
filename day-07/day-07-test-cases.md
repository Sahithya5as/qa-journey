# Day 07 — Test Cases: Metrics, RTM, Compatibility, Accessibility, i18n
**Phase:** 1 — Manual Testing Deep Dive (FINAL DAY)
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com
**Focus:** Compatibility, accessibility audit, localisation, RTM validation, exit criteria

---

## Execution Summary

| Total | Passed | Failed | Blocked |
|-------|--------|--------|---------|
| 20 | 13 | 5 | 2 |

---

## Section 1 — Cross-Browser Compatibility Tests

| TC ID | Browser | Scenario | Expected | Actual | Status |
|-------|---------|----------|----------|--------|--------|
| TC_CB_001 | Chrome v124 macOS | Login and dashboard | Works correctly | Works | ✅ Pass |
| TC_CB_002 | Firefox macOS | Login and dashboard | Works correctly | Works | ✅ Pass |
| TC_CB_003 | Safari macOS | Login and apply leave | Works correctly | Date picker shows native Safari picker — different format | ⚠️ Fail |
| TC_CB_004 | Edge Windows | Login and dashboard | Works correctly | Works | ✅ Pass |
| TC_CB_005 | Chrome mobile 375px | Login page responsive | Form usable on small screen | Usable but buttons very small — touch target too small | ⚠️ Note |
| TC_CB_006 | Chrome 768px tablet | Leave module | Works correctly | Works correctly | ✅ Pass |

---

## Section 2 — Accessibility Audit (WCAG 2.1 AA)

| TC ID | WCAG Ref | Principle | Scenario | Expected | Actual | Status |
|-------|----------|-----------|----------|----------|--------|--------|
| TC_WC_001 | 1.1.1 | Perceivable | Images have alt text | All images have descriptive alt text | Logo missing alt text | ⚠️ Fail |
| TC_WC_002 | 1.4.3 | Perceivable | Colour contrast ratio | Minimum 4.5:1 for normal text | Login button: 3.2:1 — fails AA | ⚠️ Fail |
| TC_WC_003 | 1.4.4 | Perceivable | Resize text to 200% | Content fully usable | Elements overlap at 200% | ⚠️ Fail |
| TC_WC_004 | 2.1.1 | Operable | Full keyboard navigation | All functions accessible via keyboard | Keyboard nav works | ✅ Pass |
| TC_WC_005 | 2.4.3 | Operable | Visible focus indicator | Visible focus on all interactive elements | Focus visible — blue outline | ✅ Pass |
| TC_WC_006 | 2.4.6 | Operable | Descriptive page titles | Each page has unique, descriptive title | All pages have titles | ✅ Pass |
| TC_WC_007 | 3.3.1 | Understandable | Error identification | Errors clearly describe what is wrong | Generic errors — not specific | ⚠️ Fail |
| TC_WC_008 | 4.1.2 | Robust | Form inputs have name/role/value | All form controls properly labelled | All labels correct | ✅ Pass |

---

## Section 3 — Localisation / i18n Tests (Barq Specific)
*Barq serves Saudi Arabia — Arabic RTL, Hijri calendar, Saudi Riyal critical*

| TC ID | Type | Scenario | Expected | Actual | Status |
|-------|------|----------|----------|--------|--------|
| TC_L10_001 | i18n | Arabic language switch | UI switches to RTL layout | Layout switches RTL correctly | ✅ Pass |
| TC_L10_002 | i18n | Arabic text in input fields | Arabic characters accepted in all text fields | All fields accept Arabic | ✅ Pass |
| TC_L10_003 | i18n | Date format for Arabic locale | Hijri calendar option available | Only Gregorian dates shown — no Hijri option | ⚠️ Fail |
| TC_L10_004 | i18n | Currency display | Saudi Riyal symbol displayed correctly (﷼) | Currency symbol displays correctly | ✅ Pass |
| TC_L10_005 | i18n | Right-to-left text alignment | All Arabic text aligned right | Some text fields remain LTR in Arabic mode | ⚠️ Fail |
| TC_L10_006 | l10n | All UI strings translated | No English text in Arabic mode | 3 menu items remain in English | ⚠️ Fail |

---

## Section 4 — RTM Validation (Requirements Traceability)

*Verifying 100% requirements coverage — every requirement maps to at least one test case*

| Req ID | Requirement | Test Cases | Status | Open Bugs |
|--------|------------|-----------|--------|-----------|
| REQ-001 | Admin can login with valid credentials | TC_SM_002, TC_RG_001 | ✅ Covered | — |
| REQ-002 | Login fails with invalid credentials | TC_RG_002, TC_003 | ✅ Covered | — |
| REQ-003 | Account locks after 5 failed attempts | TC_015 | ❌ Failing | BUG-001 |
| REQ-004 | Employee can apply for annual leave | TC_053, TC_RG_006 | ✅ Covered | — |
| REQ-005 | Leave balance updated after approval | TC_071, TC_072 | ✅ Covered | — |
| REQ-006 | Manager cannot approve with zero balance | TC_073 | ❌ Failing | BUG-007 |
| REQ-007 | Approved leave sends email notification | TC_075 | ✅ Covered | — |
| REQ-008 | Employee can cancel pending leave | TC_037 | ✅ Covered | — |
| REQ-009 | Application accessible via HTTPS only | TC_079 | ✅ Covered | — |
| REQ-010 | ESS users cannot access admin functions | TC_003, TC_059 | ✅ Covered | — |

**RTM Coverage: 10/10 requirements covered = 100%**
**2 requirements failing with open bugs**

---

## Section 5 — Phase 1 Final Metrics Summary

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Total test cases written | 87 | 80+ | ✅ |
| Test cases executed | 87 | 87 | ✅ |
| Pass rate | 79.3% (69/87) | ≥ 90% | ⚠️ |
| Total bugs found | 26 | — | — |
| Critical bugs | 0 | 0 | ✅ |
| High bugs open | 4 | 0 | ❌ |
| Medium bugs open | 6 | 0 | ⚠️ |
| DRE (estimated) | ~88% | ≥ 85% | ✅ |
| Requirements coverage | 100% | 100% | ✅ |
| Bug leakage (projected) | ~12% | < 5% | ❌ |

**Final Release Recommendation:** Do not release.
4 High severity bugs remain open. BUG-001 (no lockout), BUG-007 (zero balance approval), BUG-011 (deleted user login), BUG-023 (national ID exposed) — all must be fixed. Medium bugs can be deferred with product approval.


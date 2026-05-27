# Day 07 — Bug Reports: Compatibility, Accessibility, i18n Findings
**Phase:** 1 — Manual Testing Deep Dive (FINAL DAY)
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com

---

## Summary

| Bug ID | Title | Severity | Priority | Type |
|--------|-------|----------|----------|------|
| BUG-027 | Safari date picker uses native format — inconsistent with Chrome/Firefox | Low | Medium | Compatibility |
| BUG-028 | Logo image missing alt text — WCAG 1.1.1 violation | Low | Low | Accessibility |
| BUG-029 | Login button colour contrast 3.2:1 — fails WCAG 1.4.3 AA | Medium | Medium | Accessibility |
| BUG-030 | Error messages not specific — WCAG 3.3.1 failure | Medium | Medium | Accessibility |
| BUG-031 | Hijri calendar not available in Arabic locale | Medium | High | Localisation |
| BUG-032 | 3 navigation menu items not translated in Arabic mode | Medium | High | Localisation |

---

## BUG-027

**Title:** Leave Module — Safari macOS renders native date picker instead of custom picker — date format inconsistency — Safari / macOS

**Severity:** Low | **Priority:** Medium | **Type:** Compatibility

**Steps to Reproduce:**
1. Open OrangeHRM in Safari on macOS
2. Go to Leave → Apply Leave
3. Click the Start Date field

**Expected:** Same custom date picker rendered in all browsers with consistent DD/MM/YYYY format.
**Actual:** Safari renders the native browser date picker. Format shows MM/DD/YYYY — different from Chrome and Firefox. A user accustomed to DD/MM/YYYY may enter dates incorrectly in Safari.
**Impact:** Low functional impact but data entry errors possible if users switch between browsers.

---

## BUG-029

**Title:** Login page — Login button fails WCAG 1.4.3 colour contrast — measured ratio 3.2:1 against required 4.5:1 — Chrome v124 / macOS

**Severity:** Medium | **Priority:** Medium | **Type:** Accessibility
**WCAG Reference:** Success Criterion 1.4.3 — Contrast Minimum (Level AA)

**Steps to Reproduce:**
1. Run Axe DevTools browser extension on the login page
2. Or run Chrome Lighthouse → Accessibility audit
3. Observe the Login button contrast failure

**Expected:** Text to background colour contrast ratio of at least 4.5:1 per WCAG 2.1 SC 1.4.3.
**Actual:** Measured contrast ratio: 3.2:1 — fails Level AA minimum.
**Impact:** Users with low vision or colour blindness cannot clearly read the login button text. Legal compliance risk. Any organisation required to meet WCAG 2.1 AA cannot use this product in its current state.

---

## BUG-031

**Title:** OrangeHRM Arabic locale — Hijri calendar not available in date fields — only Gregorian calendar shown — Chrome v124 / macOS

**Severity:** Medium | **Priority:** High | **Type:** Localisation
**Barq Relevance:** Critical — Barq operates in Saudi Arabia where Hijri calendar is used in official documents

**Steps to Reproduce:**
1. Switch application language to Arabic
2. Go to any date field (Leave application, Employee hire date, etc.)
3. Observe the calendar picker options

**Expected:** Option to select Hijri (Islamic) calendar dates in addition to Gregorian dates. Saudi Arabian HR applications must support Hijri dates for compliance with local labour law documentation requirements.
**Actual:** Only Gregorian calendar available. No option to enter or display Hijri dates.

**Impact:**
High for Saudi market — official employment contracts, government filings, and some HR documents in Saudi Arabia require Hijri dates. Without this, the application is non-compliant for Saudi enterprise customers. Directly relevant to Barq's regulatory environment under SAMA (Saudi Central Bank).

---

## BUG-032

**Title:** OrangeHRM Arabic mode — 3 navigation menu items remain in English — incomplete translation — Chrome v124 / macOS

**Severity:** Medium | **Priority:** High | **Type:** Localisation

**Steps to Reproduce:**
1. Switch application language to Arabic
2. Observe the main navigation menu

**Expected:** All navigation menu items fully translated to Arabic when Arabic language is selected.
**Actual:** The following 3 menu items remain in English: "Recruitment", "Performance", "Buzz". All other menu items are translated. This creates a mixed-language navigation bar that looks unprofessional and confuses Arabic-speaking users.

**Impact:**
Medium — inconsistent experience for Arabic-speaking users. In a Saudi fintech product serving a majority Arabic-speaking customer base, untranslated UI strings are a product quality issue and potential regulatory compliance gap.

---

## Phase 1 Complete — All Bug Reports Summary

| Phase 1 Total Bugs | 32 |
|--------------------|-----|
| Critical | 0 |
| High | 6 |
| Medium | 18 |
| Low | 8 |
| Security bugs | 5 |
| Accessibility bugs | 5 |
| Localisation bugs | 3 |
| Business logic bugs | 8 |
| UX bugs | 6 |
| Compatibility bugs | 5 |


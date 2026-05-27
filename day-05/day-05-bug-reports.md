# Day 05 — Bug Reports: Exploratory + Gorilla Testing Findings
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome v124
**URL:** https://opensource-demo.orangehrmlive.com

---

## Summary

| Bug ID | Title | Severity | Priority | Found By |
|--------|-------|----------|----------|----------|
| BUG-017 | Leave accepted on public holidays — counted as working days | Medium | Medium | Exploratory |
| BUG-018 | Leave type with zero entitlement remains selectable | Low | Low | Exploratory |
| BUG-019 | Page refresh loses all leave form data — no warning | Medium | Medium | Exploratory |
| BUG-020 | Employee comments not visible to manager in approval view | Medium | High | Exploratory |
| BUG-021 | Blank screen shown when JavaScript is disabled | Low | Low | Gorilla |
| BUG-022 | Login form overlaps at 200% zoom — WCAG violation | Medium | Medium | Gorilla |

---

## BUG-017

**Title:** Leave Module — System allows leave on public holidays and counts them as working days — no public holiday calendar integration — Chrome v124 / macOS

**Severity:** Medium | **Priority:** Medium | **Found By:** Exploratory testing

**Steps to Reproduce:**
1. Login as Admin → Go to Leave → Apply Leave
2. Select dates that include a public holiday
3. Submit the leave request

**Expected:** Public holidays excluded from leave count OR warning shown: "Your dates include X public holidays — these will not be deducted from your balance."
**Actual:** Public holidays counted as regular working days. Leave balance incorrectly deducted for non-working days.
**Impact:** Incorrect leave balance calculations. Financial impact on employees whose leave is incorrectly deducted.

---

## BUG-019

**Title:** Leave Module — All data lost on page refresh during leave application — no unsaved changes warning — Chrome v124 / macOS

**Severity:** Medium | **Priority:** Medium | **Found By:** Exploratory testing

**Steps to Reproduce:**
1. Go to Leave → Apply Leave
2. Fill in all fields — leave type, start date, end date, write a long comment
3. Press Cmd+R to refresh before submitting

**Expected:** Warning dialog: "You have unsaved changes. Are you sure you want to leave?" OR auto-save preserves data.
**Actual:** All data lost immediately with no warning. User must re-enter everything from scratch.
**Impact:** Poor user experience. Real-world users often accidentally refresh, especially on slow connections.

---

## BUG-020

**Title:** Leave Module — Employee comments entered during leave application are not visible to manager in approval view — Chrome v124 / macOS

**Severity:** Medium | **Priority:** High | **Found By:** Exploratory testing

**Steps to Reproduce:**
1. Login as employee → Apply Leave → Enter comment: "Family emergency, need urgent approval"
2. Submit
3. Login as Admin → Leave List → Open the pending request

**Expected:** Comment clearly visible in approval view so manager can make informed decision.
**Actual:** Comment field completely absent from manager's approval view. Manager sees only name, leave type, and dates — not the reason.
**Impact:** High — managers approving/rejecting without knowing the reason defeats the entire purpose of the comment field. Misleads employees into thinking their reason will be read.

---

## BUG-022

**Title:** Login page — Form elements overlap and become unusable at 200% browser zoom — WCAG 2.1 SC 1.4.4 violation — Chrome v124 / macOS

**Severity:** Medium | **Priority:** Medium | **Found By:** Gorilla testing

**Steps to Reproduce:**
1. Open Chrome → Navigate to app
2. Press Cmd + repeatedly to zoom to 200%
3. Observe the login form

**Expected:** Form fully usable at 200% zoom per WCAG 2.1 Success Criterion 1.4.4 (Level AA).
**Actual:** Username label overlaps input field. Password field partially hidden behind Login button. Forgot Password link requires horizontal scrolling to find.
**WCAG Reference:** Success Criterion 1.4.4 — Resize Text (Level AA)
**Impact:** Users with visual impairments relying on browser zoom cannot use the login page. Legal compliance risk for organisations required to meet WCAG AA.


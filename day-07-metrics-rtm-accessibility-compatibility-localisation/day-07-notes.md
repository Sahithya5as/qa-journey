# Day 07 — Test Metrics, RTM, Test Closure, Compatibility, Accessibility
**Phase:** 1 — Manual Testing Deep Dive (FINAL DAY)
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com

---

## 1. Test Metrics — Complete Guide

Test metrics are the numbers that tell you the health of your testing effort. Senior QAs present these to management every sprint. Know every metric, its formula, and what it tells you.

### The 7 Core Metrics

**1. Defect Removal Efficiency (DRE)**
```
DRE = (Bugs found before release ÷ Total bugs) × 100
Total bugs = Pre-release bugs + Post-release bugs
```
Example: 90 bugs found in testing + 10 found in production = DRE of 90%
World-class target: 95%+. Industry benchmark: 85%+.
What it tells you: How effective your QA process is at catching bugs before users do.

**2. Defect Leakage Rate**
```
Leakage % = (Bugs found in production ÷ Total bugs) × 100
```
Example: 10 production bugs out of 100 total = 10% leakage
Target: below 5%. What it tells you: How many bugs escaped your QA process.
Action when high: Review what the production bugs had in common — missing test coverage area.

**3. Bug Density**
```
Bug Density = Total bugs found ÷ Module size (KLOC or per feature)
```
What it tells you: Which modules are most defect-prone. Use to direct testing effort.
In practice: "The Leave module had the highest bug density this sprint — 4.2 bugs per feature. Recommending additional unit test coverage."

**4. Test Execution Rate**
```
Execution Rate = (Test cases executed ÷ Total test cases planned) × 100
```
Target: 95%+ before any release. What it tells you: Are you on track with testing?

**5. Test Pass Rate**
```
Pass Rate = (Test cases passed ÷ Test cases executed) × 100
```
Healthy release: 90%+ pass rate with all failures investigated and either fixed or formally accepted.

**6. Defect Fix Rate**
```
Fix Rate = (Bugs fixed ÷ Total bugs raised) × 100
```
What it tells you: How fast development is resolving defects.

**7. Reopen Rate**
```
Reopen Rate = (Reopened bugs ÷ Total fixed bugs) × 100
```
Target: below 5%. What a high reopen rate tells you: Developers are marking bugs fixed without properly testing their fix. Action: Introduce a developer self-test checklist before any bug is marked Fixed.

**8. Mean Time to Detect (MTTD)**
Average time from when a bug was introduced to when QA detected it.
Lower is better. Reduced by shift-left testing and frequent builds.

**9. Mean Time to Resolve (MTTR)**
Average time from bug report to verified fix.
Lower is better. Reduced by clear bug reports and responsive development team.

### Presenting Metrics to Management

Never just present numbers — present insights and actions.

**Junior QA:** "We found 45 bugs this sprint."

**Senior QA:** "We found 45 bugs this sprint — 3 Critical, 8 High, 22 Medium, 12 Low. DRE stands at 87% — slightly below our 90% target. The majority of bugs were clustered in the payment flow module which had the most code churn this sprint. I am recommending we add 15 additional regression cases to the payment module for next sprint to close this gap. Bug leakage from last sprint was 3% — within our target."

---

## 2. RTM — Requirement Traceability Matrix

The RTM is a document that maps every requirement to its corresponding test cases. It ensures nothing is missed — every requirement has at least one test case, and every test case traces back to a requirement.

### RTM Structure

| Req ID | Requirement Description | Test Case IDs | Execution Status | Defect IDs |
|--------|------------------------|---------------|-----------------|------------|
| REQ-001 | Employee can login with valid credentials | TC_001, TC_002 | Pass | — |
| REQ-002 | Login fails with invalid credentials | TC_003, TC_004, TC_005 | Pass | — |
| REQ-003 | Account locks after 5 failed attempts | TC_015 | Fail | BUG-001 |
| REQ-004 | Employee can apply for annual leave | TC_021, TC_022, TC_023 | Pass | — |
| REQ-005 | Leave balance updated after approval | TC_071, TC_072 | Fail | BUG-007 |
| REQ-006 | Manager receives email on leave request | TC_075 | Pass | — |

### Why RTM matters

**For QA:** Proves 100% requirements coverage. No requirement left untested.
**For management:** Shows exactly which requirements are working, which have bugs, and which are not yet tested.
**For audit:** Regulatory compliance requires traceability — fintech companies must demonstrate that every requirement was tested and either passed or has a documented defect.

### RTM in Agile
In Agile, requirements are user stories with acceptance criteria. The RTM maps each AC to test cases.

| Story ID | Acceptance Criteria | Test Cases | Status | Bugs |
|----------|--------------------|-----------|----|------|
| STORY-001 | AC1: Valid login succeeds | TC_001 | ✅ Pass | — |
| STORY-001 | AC2: Invalid login shows error | TC_002, TC_003 | ✅ Pass | — |
| STORY-001 | AC3: 5 failed attempts locks account | TC_015 | ❌ Fail | BUG-001 |

---

## 3. Test Summary Report

Written at the end of each test cycle. Shorter than the closure report. The key document for the release decision.

### Test Summary Report Template

**1. Executive Summary**
One paragraph — scope, overall result, release recommendation.
Example: "Testing of OrangeHRM Leave Management v2.1 is complete. 87 of 90 test cases passed (96.7%). 3 open defects remain — 0 Critical, 1 High, 2 Medium. Recommendation: Release approved with BUG-019 and BUG-020 deferred. BUG-007 must be fixed before release."

**2. Test Scope**
What was tested and what was out of scope.

**3. Test Execution Summary**

| Metric | Value |
|--------|-------|
| Total test cases planned | 90 |
| Total executed | 90 |
| Passed | 83 |
| Failed | 5 |
| Blocked | 2 |
| Execution rate | 100% |
| Pass rate | 92.2% |

**4. Defect Summary**

| Severity | Raised | Fixed | Open | Deferred |
|----------|--------|-------|------|----------|
| Critical | 0 | 0 | 0 | 0 |
| High | 3 | 2 | 1 | 0 |
| Medium | 8 | 6 | 0 | 2 |
| Low | 5 | 3 | 0 | 2 |
| **Total** | **16** | **11** | **1** | **4** |

**5. Risk Assessment**
Open bugs and their business impact. What is the risk of releasing with each open item.

**6. Go/No-Go Recommendation**
Clear recommendation with justification.

**7. Lessons Learned**
What went well, what did not, what to do differently next sprint.

---

## 4. Test Closure Report

Written at the end of the entire project — not just one sprint. More comprehensive than the test summary report.

**Additional sections in Test Closure Report:**
- Metrics trends across all sprints (DRE trend, defect density trend)
- Test environment issues encountered
- Process improvements implemented during the project
- Recommendations for the next project or next release
- Sign-off signatures from QA lead, product manager, project manager

---

## 5. Sign-off Process

Before any production release, formal sign-off is required from:
- **QA Lead:** All exit criteria met, test summary report approved
- **Product Manager:** Business requirements validated, open bugs reviewed and approved for deferral
- **Project Manager:** Schedule, resources, and risk acknowledged
- **Development Lead:** All fixed bugs verified, no known regressions

**QA's role:** Prepare the sign-off document, present the evidence, answer questions, and formally record the decision and who made it.

---

## 6. Localisation and Internationalisation Testing

### Internationalisation (i18n)
Testing that the application is built to support multiple languages and regions without code changes. The number 18 = 18 letters between i and n in "internationalisation."

**What to test:**
- Does the app handle Unicode characters? (Arabic, Chinese, Hindi)
- Do date formats work for different regions? (DD/MM/YYYY vs MM/DD/YYYY)
- Do currency formats display correctly? ($ vs £ vs ﷼ for Saudi Riyal — Barq relevant)
- Does the layout handle right-to-left (RTL) languages? (Arabic — highly relevant for Barq)
- Do phone number formats work for different countries?
- Do postal code formats work globally?

### Localisation (l10n)
Testing that the application has been correctly translated and culturally adapted for a specific region. The 10 = 10 letters between l and n.

**What to test:**
- Are all UI strings translated? No English text appearing in Arabic mode?
- Are translations accurate and contextually correct?
- Do images and icons make sense culturally?
- Are legal and compliance texts correct for the local market?
- Does the app comply with local data privacy laws?

**Barq-specific relevance:**
Barq is a Saudi fintech serving 85+ nationalities. Arabic RTL layout, Saudi Riyal currency, Hijri calendar dates, and SAMA (Saudi Central Bank) compliance are all localisation test areas critical for this role.

---

## 7. Complete Phase 1 — Advanced Terminology Reference

Every term you need to know cold for any senior QA interview:

| Term | Definition |
|------|-----------|
| Shift-Left | Moving testing earlier in SDLC |
| Shift-Right | Testing in production — canary, A/B, monitoring |
| Three Amigos | BA + Dev + QA meeting before development |
| Entry Criteria | Conditions before testing can start |
| Exit Criteria | Conditions before release can proceed |
| Risk-Based Testing | Prioritise by probability × impact |
| DRE | Defect Removal Efficiency — % bugs caught pre-release |
| Bug Leakage | Bugs that escaped to production |
| Bug Density | Bugs per unit of code/feature |
| MTTR | Mean Time to Resolve |
| MTTD | Mean Time to Detect |
| RTM | Requirement Traceability Matrix |
| Stub | Fake dependency returning hardcoded data |
| Mock | Fake dependency that also verifies it was called correctly |
| Driver | Test code that calls a component when its caller is not built |
| Test Harness | Complete set of test components assembled for testing |
| TDD | Test-Driven Development — write test before code |
| BDD | Behaviour-Driven Development — Gherkin Given/When/Then |
| ATDD | Acceptance Test-Driven Development |
| RCA | Root Cause Analysis — 5 Whys technique |
| Definition of Done | Checklist that must be true before story is complete |
| Acceptance Criteria | Specific testable conditions for a user story |
| Canary Release | Deploy to % of users first, monitor, then expand |
| Blue-Green | Two identical environments — instant traffic switch |
| Shadow Testing | Mirror production traffic to new system for comparison |
| Fail Fast | Identify failures early — stop the pipeline immediately |
| Smoke Test | Quick check — is the build stable? |
| Sanity Test | Quick check — is this specific fix working? |
| Regression | Full check — did new code break existing features? |
| Retesting | Exact rerun — is this specific bug fixed? |
| Exploratory | Creative investigation — find unknown bugs |
| Gorilla | Exhaustive testing of one feature |
| Monkey | Random testing to find crashes |
| UAT | Business users validate the software |
| Alpha Testing | Internal pre-release testing |
| Beta Testing | External real-user pre-release testing |
| SBET | Session-Based Exploratory Testing |
| SFDIPOT | Heuristic for exploratory testing scope |
| WCAG | Web Content Accessibility Guidelines |
| i18n | Internationalisation — built to support multiple regions |
| l10n | Localisation — adapted for a specific region |
| p95 latency | 95% of requests complete within this response time |
| Apdex | User satisfaction score for performance |
| OWASP Top 10 | 10 most critical web security vulnerabilities |
| Horizontal escalation | User A accessing User B's data (same level) |
| Vertical escalation | Regular user accessing admin functions |
| Secrets management | Preventing API keys/tokens from being exposed |
| Idempotency | Same request executed multiple times produces same result |
| Eventual consistency | Distributed system data converges to same state over time |

---

## Phase 1 Complete — What You Now Know

After 7 days of Phase 1 you have covered:
- Complete QA fundamentals and mindset
- All test design techniques with examples
- Full bug lifecycle and defect metrics with formulas
- Test planning, strategy, entry/exit criteria
- All testing types — functional and non-functional
- All advanced terminology
- Agile QA — all Scrum ceremonies
- All test metrics and how to present them
- RTM — structure and purpose
- Test summary report and closure report
- Compatibility, accessibility, localisation
- Stubs, mocks, drivers, test harness
- TDD, BDD, ATDD, RCA
- Shift-left, shift-right, risk-based testing

**You are now ready to speak confidently about manual QA at a senior level in any interview.**


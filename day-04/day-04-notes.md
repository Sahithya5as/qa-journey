# Day 04 — Test Planning, Test Strategy, Shift-Left, Risk-Based Testing
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com

---

## 1. Test Plan vs Test Strategy — The Difference Most People Get Wrong

Most people use these terms interchangeably in interviews. They are completely different documents with different owners, audiences, and purposes.

### Test Strategy
High-level document defining the overall testing approach for the entire organisation or project. Written once, rarely changes. Answers: **"How does this organisation approach testing?"**

**Written by:** QA Manager or Test Architect
**Audience:** Senior management, architects, QA leads
**Scope:** Organisation-wide or programme-wide
**Lifespan:** Long-term

**Contains:**
- Testing objectives and goals
- Types of testing to be performed
- Tools and frameworks to be used
- Test environment strategy
- Defect management approach
- Risk management approach
- Roles and responsibilities
- Entry and exit criteria standards
- Metrics and reporting standards

### Test Plan
Detailed, living document specific to one project or release. Updated throughout the testing cycle. Answers: **"How will we test THIS release?"**

**Written by:** QA Lead or Senior QA Engineer
**Audience:** QA team, developers, project manager, stakeholders
**Scope:** One project or one sprint
**Lifespan:** Duration of the testing cycle

### Anatomy of a Test Plan — All 11 Sections

**1. Introduction and Objectives**
What is being tested and why.
Example: "This test plan covers OrangeHRM Leave Management v2.1 to validate all requirements in JIRA Epic LM-001."

**2. Scope**
In scope: what will be tested.
Out of scope: what will NOT be tested and why — prevents stakeholders expecting untested coverage.

**3. Test Approach**
How testing will be conducted — techniques, levels, types.
Example: "Manual black-box testing using EP and BVA for functional testing. Exploratory testing for edge cases. Automated regression using Playwright."

**4. Test Environment**
Server details, browsers, devices, test data sources.
Example: "Testing on staging.orangehrm.com using Chrome v124 on macOS. Test data created fresh each cycle."

**5. Entry Criteria**
Conditions that must be met BEFORE testing begins. If not met — testing does not start.
Standard entry criteria:
- Build deployed to test environment
- All unit tests passing — no broken build
- Test data set up and verified
- Test cases reviewed and approved
- All credentials and access available

**6. Exit Criteria**
Conditions that must be met BEFORE release can proceed. QA's quality gate.
Standard exit criteria:
- 95%+ test cases executed
- Zero open Critical or High severity bugs
- All Medium bugs fixed or formally deferred
- Regression suite passing
- Test summary report approved
- Sign-off received from product manager

**7. Resource Plan**
Who does what and when.

**8. Schedule**
Timeline for each testing activity.

**9. Risk and Mitigation**

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Test environment goes down | Medium | High | Backup environment + escalation contact |
| Developer unavailable for critical fixes | Low | High | Agree fix SLA upfront |
| Requirements change mid-cycle | Medium | High | Freeze requirements 3 days before cycle |
| Test data corrupted | Low | Medium | Daily backups of test database |

**10. Deliverables**
What QA will produce — test cases, bug reports, execution report, summary report.

**11. Approvals**
Who signs off the test plan before testing begins.

---

## 2. Entry and Exit Criteria — Deep Dive

### Entry Criteria — Full Checklist
Before starting any test cycle verify:
- [ ] Build number confirmed and deployed
- [ ] Release notes received — what changed in this build
- [ ] All unit tests passing in CI pipeline
- [ ] Test environment stable and accessible
- [ ] Test data set up — users, accounts, reference data ready
- [ ] Test cases written and peer-reviewed
- [ ] All third-party dependencies confirmed available
- [ ] Browser and device matrix confirmed

**If entry criteria not met:** Document it formally. Never start testing on an unstable environment — it corrupts your metrics and wastes time.

### Exit Criteria — The Release Decision Framework
What you bring to the exit criteria meeting:
- Test execution report: X executed, Y passed, Z failed, W blocked
- Open bug summary: X Critical, Y High, Z Medium, W Low — all with status
- Risk assessment: business impact of releasing with known open bugs
- Recommendation: release / release with conditions / do not release

**Senior QA in a release meeting:**
"We have 2 open High severity bugs. BUG-007 is the leave balance enforcement bug — direct payroll impact, recommend blocking. BUG-008 is the missing feedback on cancelled leave — annoying but not business-critical, recommend deferring with a known issue note."

---

## 3. Shift-Left Testing

### What it means
Moving testing activities earlier in the development lifecycle — toward the left of the timeline.

```
Traditional:  [Requirements] → [Design] → [Development] → [TESTING] → [Release]
Shift-Left:   [Requirements] → [Design] → [Development]             → [Release]
               ↑ QA here        ↑ QA here    ↑ QA here
```

### Cost of finding bugs late

| When bug found | Relative cost to fix |
|----------------|---------------------|
| During requirements review | 1x |
| During design review | 5x |
| During development | 10x |
| During testing | 20x |
| After release in production | 100x |

### What QA does in shift-left

**During requirements:**
- Review user stories for testability
- Flag ambiguous requirements — "validate correctly" is not testable
- Write acceptance criteria with the BA
- Identify missing requirements before development starts

**During design:**
- Flag design decisions that make testing difficult
- Identify test data requirements early
- Review API contracts before they are built

**During development:**
- Write test cases while development is in progress
- Set up test environment in parallel
- Pair with developers on complex features

### Three Amigos
BA + Developer + QA meet before a feature is built.
- BA brings the business requirement
- Developer brings the implementation approach
- QA brings the test scenarios and edge cases
Result: shared understanding before a single line of code is written.

**Interview answer:**
"I actively participate in three amigos sessions, review user stories for testability, write test cases in parallel with development, and flag requirement gaps before they become code defects. A bug found in requirements costs almost nothing — the same bug in production costs 100x more."

---

## 4. Shift-Right Testing

Testing that happens after release, in production.

| Activity | What it is |
|----------|-----------|
| Canary Release | Deploy to 5% of users first, monitor, then expand gradually |
| Blue-Green Deployment | Two identical environments — instant traffic switch, instant rollback |
| Shadow Testing | Mirror production traffic to new system — compare responses — zero user impact |
| A/B Testing | Two versions shown to different user groups — measure which performs better |
| Production Monitoring | Watch Grafana, Sentry, Datadog dashboards after every deployment |

---

## 5. Risk-Based Testing

With limited time and unlimited things to test — prioritise by risk.

```
Risk = Probability of failure × Impact of failure
```

| Feature | Probability | Impact | Risk | Priority |
|---------|-------------|--------|------|----------|
| Payment processing | High | High | 🔴 High | Test first, test thoroughly |
| Leave approval | Medium | High | 🔴 High | Test thoroughly |
| Profile photo upload | Low | Low | 🟢 Low | Smoke test only |
| Dashboard colour theme | Low | Low | 🟢 Very Low | Skip or test last |

### Factors that increase probability
- Area recently changed
- Complex business logic
- Third-party integration
- Known bug history in this area
- New developer working on it

### Factors that increase impact
- Many users affected
- Financial transactions involved
- Data loss possible
- Security involved
- No workaround available

**Interview answer:**
"I look at what changed in the build, assess each area for probability and business impact, and allocate testing time accordingly. High risk areas get full coverage. Low risk areas get smoke tested. I document my risk assessment so the team understands what was tested thoroughly and what was covered lightly."

---

## 6. Test Coverage Types

| Type | What it measures |
|------|-----------------|
| Requirements Coverage | Are all requirements covered by at least one test case? Tracked via RTM. |
| Code Coverage | What % of code is executed during testing — statement, branch, path coverage |
| Functional Coverage | Are all features and user journeys covered? |
| Risk Coverage | Are the highest-risk areas covered by the most thorough tests? |

---

## 7. Advanced Terminology

| Term | Definition |
|------|-----------|
| Shift-Left | Moving testing earlier in SDLC — catch bugs when cheapest to fix |
| Shift-Right | Testing in production — canary, monitoring, A/B testing |
| Three Amigos | BA + Dev + QA meeting before development begins |
| Entry Criteria | Conditions that must be met before testing starts |
| Exit Criteria | Conditions that must be met before release proceeds |
| Risk-Based Testing | Prioritising test effort by probability × impact |
| Canary Release | Deploy to small % of users first, monitor, then expand |
| Blue-Green Deployment | Two identical environments — instant traffic switch |
| Shadow Testing | Mirror production traffic to new system for comparison |
| Fail Fast | Identify and report failures as early as possible |
| DRE | Defect Removal Efficiency — % of bugs caught before production |
| Test Oracle | Source of truth for determining pass/fail of a test |

---

## Interview Talking Points

- Can explain the full anatomy of a Test Plan — all 11 sections and why each exists
- Know the difference between Test Plan and Test Strategy — who writes each, when, scope
- Can define entry and exit criteria and use them in a real release decision
- Can explain shift-left with concrete examples at each SDLC stage
- Know the Three Amigos and can describe running one
- Apply risk-based testing using probability × impact framework
- Know all four test coverage types and when each is relevant


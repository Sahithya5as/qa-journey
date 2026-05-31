# Day 06 — Non-Functional Testing + Advanced Terminology + TDD/BDD/ATDD + Agile QA
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com

---

## 1. Non-Functional Testing — Complete Guide

Non-functional testing validates HOW the system performs — not WHAT it does. Functional testing checks features. Non-functional testing checks quality attributes.

### Performance Testing
Evaluates how the system behaves under expected and unexpected load conditions.

**Key metrics every QA must know:**

| Metric | Definition | Good target |
|--------|-----------|-------------|
| Response time | Time from request to response | < 2 seconds for web |
| Throughput | Requests processed per second (RPS/TPS) | Depends on system |
| Error rate | % of requests that fail | < 1% under normal load |
| p50 latency | 50% of requests complete within this time | Baseline measurement |
| p95 latency | 95% of requests complete within this time | Real-world user experience |
| p99 latency | 99% of requests complete within this time | Worst-case scenario |
| Apdex score | User satisfaction score 0–1 | > 0.85 |

**Why p95 matters more than average:**
Average response time can look healthy while 5% of users experience 10-second timeouts. p95 tells you the real user experience at the edges.

### Load Testing
Testing under expected normal and peak load conditions to validate the system performs within acceptable thresholds.

**Scenarios:**
- Normal load: typical daily traffic
- Peak load: maximum expected traffic (e.g. Black Friday, salary payment day)
- Goal: confirm system meets SLAs under these conditions

### Stress Testing
Pushing the system beyond its limits to find the breaking point. Deliberately overwhelming the system.

**Goal:** Find where the system breaks, how it fails, and whether it recovers gracefully.
**Key question:** Does it fail gracefully (clean error message) or catastrophically (data corruption, crash)?

### Spike Testing
Sudden, extreme increase in load — simulating a traffic spike. E.g. a viral post sends 10x normal traffic in 30 seconds.

**Goal:** Does the system handle sudden spikes without crashing? Does it auto-scale?

### Soak Testing (Endurance Testing)
Running the system at normal load for an extended period — hours or days.

**Goal:** Find memory leaks, database connection pool exhaustion, disk space issues that only appear over time.

### Scalability Testing
Testing how the system scales up (vertical — bigger server) or out (horizontal — more servers).

**Goal:** Confirm the system can grow with increasing user base.

### Performance Testing in Fintech Context (Barq relevance)
In a payment system, performance testing is critical because:
- Slow payment confirmation = customer distrust
- Failed transactions under load = revenue loss
- p99 latency for payment API must be under 3 seconds — regulatory requirement in some markets
- Spike testing for salary payment days — first of month sees 10x normal transaction volume

---

## 2. Security Testing

Testing the system for vulnerabilities, threats, and risks to protect data and prevent unauthorised access.

### OWASP Top 10 — Must Know for Every Senior QA

| # | Vulnerability | What it is | How QA tests it |
|---|--------------|-----------|-----------------|
| 1 | Injection (SQLi) | Malicious SQL injected into input fields | Enter ' OR '1'='1 in all input fields |
| 2 | Broken Authentication | Weak session management, exposed tokens | Test session expiry, token reuse, account lockout |
| 3 | Sensitive Data Exposure | PII/passwords in logs, URLs, responses | Check network traffic, response bodies, logs |
| 4 | XML External Entities (XXE) | Malicious XML processed by the server | Send XXE payloads in XML inputs |
| 5 | Broken Access Control | User A can access User B's data | Change user IDs in requests — horizontal escalation |
| 6 | Security Misconfiguration | Debug headers, verbose error messages | Check response headers, error messages |
| 7 | XSS (Cross-Site Scripting) | Malicious script injected into web page | Enter <script>alert(1)</script> in all text fields |
| 8 | Insecure Deserialization | Malicious objects passed to the application | Send modified serialized objects |
| 9 | Using Components with Known Vulnerabilities | Outdated libraries with known CVEs | Check dependency versions |
| 10 | Insufficient Logging | No audit trail of actions | Verify all critical actions are logged |

### Authentication vs Authorisation
- **Authentication:** Are you who you say you are? (Login)
- **Authorisation:** Are you allowed to do what you are trying to do? (Permissions)

**Horizontal privilege escalation:** User A accessing User B's data (same permission level)
**Vertical privilege escalation:** Regular user accessing admin functionality

### Secrets Management Testing
In API responses and logs, check:
- API keys not exposed in response body
- Passwords not in plain text in logs
- Tokens not visible in browser URL
- Internal server paths not exposed in error messages

---

## 3. Accessibility Testing

Testing that the application is usable by people with disabilities.

### WCAG 2.1 — Four Principles (POUR)

| Principle | Meaning | Examples |
|-----------|---------|---------|
| Perceivable | Users can perceive all content | Alt text on images, captions on videos |
| Operable | Users can operate all UI components | Keyboard navigation, no seizure-inducing content |
| Understandable | Users can understand content | Clear error messages, consistent navigation |
| Robust | Content works with assistive technologies | Works with screen readers, proper semantic HTML |

### WCAG Conformance Levels
- **Level A:** Minimum — must meet
- **Level AA:** Standard — most organisations target this
- **Level AAA:** Highest — difficult to achieve for all content

### Tools
- **Axe:** Browser extension — automated accessibility scan
- **Wave:** Visual accessibility evaluation
- **Lighthouse:** Chrome DevTools built-in audit
- **NVDA:** Free screen reader for Windows
- **VoiceOver:** Built-in screen reader on Mac/iOS

---

## 4. Compatibility Testing

Testing that the application works correctly across different environments.

### Cross-browser testing
Chrome, Firefox, Safari, Edge — different rendering engines produce different behaviour.

### Cross-device testing
Desktop, tablet, mobile — different screen sizes, touch vs mouse, different hardware.

### Responsive vs Adaptive Design
- **Responsive:** One layout that fluidly adjusts to any screen size (CSS media queries)
- **Adaptive:** Multiple fixed layouts — system detects device and serves the right one

### CSR vs SSR Testing
- **CSR (Client-Side Rendering):** JavaScript builds the page in the browser — test with JS disabled
- **SSR (Server-Side Rendering):** Server sends complete HTML — faster initial load, test for hydration issues

### PWA/SWA Testing
- Test offline mode — does the app work without internet?
- Test service worker caching — does the app show stale data?
- Test push notifications (PWA)
- Test add-to-homescreen behaviour

### Browser addons for QA
- **Bug Magnet:** Fills fields with edge case data (SQL injection strings, long strings, special chars) in one click
- **Ghost Inspector:** Records and replays test sessions
- **Selenium IDE:** Record and playback browser actions

---

## 5. Stubs, Mocks, Drivers, and Test Harness

These are test doubles — objects that stand in for real components during testing.

### Stub
A simplified implementation of a dependency that returns hardcoded responses. Used when the real dependency is unavailable or too complex to set up.

**Example:** Payment gateway is not available in the test environment. A stub returns a hardcoded "payment successful" response so you can test the downstream flow without a real payment.

### Mock
Similar to a stub but also verifies that the dependency was called correctly — the right method, the right number of times, with the right parameters.

**Example:** You want to verify that when an employee is approved for leave, the email service is called exactly once with the correct recipient email.

### Stub vs Mock

| | Stub | Mock |
|--|------|------|
| Purpose | Returns fake data | Returns fake data AND verifies calls |
| Verification | No | Yes — was it called? How many times? With what params? |
| Use when | Testing flow when dependency unavailable | Testing that your code interacts with dependency correctly |

### Driver
A piece of test code that calls the component being tested when the caller (the module above it) is not yet built. Used in bottom-up integration testing.

**Example:** The UI is not built yet but the API is ready. A driver simulates the UI calling the API.

### Test Harness
A collection of all test components — test cases, drivers, stubs, mocks, test data, test scripts — assembled together to enable automated testing of a component.

---

## 6. TDD, BDD, ATDD, RCA

### TDD — Test-Driven Development
Developer writes the test BEFORE writing the code. The test fails first (Red), then the developer writes code to make it pass (Green), then refactors (Refactor). This is the Red-Green-Refactor cycle.

**QA's role in TDD:** QA does not write unit tests — that is the developer's job. But QA should understand TDD because it affects how testable the code is. TDD-practised code is generally cleaner and easier to test at all levels.

### BDD — Behaviour-Driven Development
An extension of TDD where tests are written in plain English using the Gherkin syntax — Given/When/Then — so that non-technical stakeholders can read and contribute to test scenarios.

**Gherkin syntax:**
```gherkin
Feature: Leave Application

  Scenario: Employee applies for leave with sufficient balance
    Given the employee has 5 days of annual leave remaining
    When the employee applies for 2 days of leave
    Then the leave request should be created with status "Pending"
    And the employee's remaining balance should show 3 days

  Scenario: Employee cannot apply for leave with zero balance
    Given the employee has 0 days of annual leave remaining
    When the employee attempts to apply for 1 day of leave
    Then the system should display "Insufficient leave balance"
    And no leave request should be created
```

**Tools:** Cucumber (Java/JavaScript), Behave (Python), SpecFlow (.NET)

**Benefits:**
- Living documentation — scenarios stay current with the code
- Shared language between BA, Dev, and QA
- Business stakeholders can read and validate scenarios

### ATDD — Acceptance Test-Driven Development
Similar to BDD but focuses on acceptance criteria defined by the customer/business before development starts. The acceptance tests become the definition of "done."

**Difference from BDD:**
- BDD focuses on behaviour and uses Gherkin
- ATDD focuses on acceptance criteria and can use any format
- In practice many teams use both terms interchangeably

### RCA — Root Cause Analysis
A structured process for identifying the fundamental cause of a problem — not just the symptom.

**Most common RCA technique — 5 Whys:**

**Problem:** Payment API is returning 500 errors in production.
- Why? → The database is returning timeout errors
- Why? → The query is taking too long to execute
- Why? → There is no index on the user_id column
- Why? → The schema was updated last sprint without index review
- Why? → There is no database change review process in the team

**Root cause:** No database change review process
**Fix:** Implement a database change review checklist in the PR process

**QA's role in RCA:**
In a fintech like Barq, QA participates in incident RCA. When a production issue occurs, QA provides:
- Timeline of when the issue was first detected
- Test evidence from the last passing test cycle
- Analysis of whether the issue was in scope and why it was not caught
- Recommendations for preventing recurrence in the test process

---

## 7. Agile Testing — Scrum Ceremonies for QA

### Sprint Planning
QA's role: review user stories for testability, estimate testing effort, flag stories with missing acceptance criteria, identify stories that need three amigos sessions.

**QA questions to ask in sprint planning:**
- "This story has no acceptance criteria — we cannot test it as written"
- "This story depends on the payment gateway — is that available in our test environment?"
- "This story touches the authentication module — I need to run regression on login after this"

### Sprint Refinement (Backlog Grooming)
QA reviews upcoming stories before they enter the sprint. Earlier than sprint planning — catches requirement issues earlier.

**QA's contribution:** Raise questions, define acceptance criteria, identify dependencies, estimate test complexity.

### Daily Standup
QA reports: what was tested yesterday, what will be tested today, any blockers (build broken, environment down, missing test data).

**Senior QA standup:** "Yesterday I completed regression on the leave module — 45 cases passed, 2 failed — BUG-019 and BUG-020 raised. Today I am starting the payment flow testing. Blocked on the payment gateway credentials for staging — need from DevOps."

### Sprint Review (Demo)
QA participates by demonstrating that the acceptance criteria are met. Can show test execution reports.

### Sprint Retrospective
QA contributes to: what went well in testing this sprint, what did not, what process improvements to make next sprint.

**Senior QA retro contribution:** "We had 3 Cannot Reproduce bugs this sprint — recommend we agree on a screen recording requirement before any bug is raised."

### Definition of Done (DoD)
A shared checklist that must be true before any story is considered complete. QA owns the testing-related criteria.

**Example DoD items owned by QA:**
- All acceptance criteria have test cases written
- Test cases executed and passing
- No open Critical or High bugs
- Regression suite updated to include new test cases
- Test summary report updated

### Acceptance Criteria
Conditions that must be met for a user story to be considered done. Written by BA with input from QA. Must be specific, testable, and unambiguous.

**Bad AC:** "The system should validate the form correctly"
**Good AC:** "When the user submits the leave application form with an empty start date field, the system should display the error message 'Start date is required' below the start date field and the form should not be submitted"

---

## 8. Fail Fast

A principle where failures are identified and reported as early as possible in the development and deployment pipeline, rather than letting them propagate.

**In CI/CD:** Tests run on every commit. If tests fail, the pipeline stops immediately and the developer is notified. The code does not proceed to the next stage.

**In Agile:** QA tests new features as soon as they are available — not at the end of the sprint. Daily testing prevents a pile-up of bugs at sprint end.

**Benefits:**
- Cheaper to fix early
- Shorter feedback loop
- Prevents blocking other team members on broken code

---

## Interview Talking Points

- Can explain all non-functional testing types with fintech-specific examples
- Know OWASP Top 10 and how to test each vulnerability manually
- Know the difference between authentication and authorisation — horizontal vs vertical escalation
- Can explain stub vs mock with a real example from payment or leave system
- Know the TDD Red-Green-Refactor cycle
- Can write Gherkin scenarios for any feature from scratch
- Know the difference between BDD and ATDD
- Can run a 5 Whys RCA on any production incident
- Can articulate QA's exact role in every Scrum ceremony
- Know Definition of Done and can write QA-owned criteria
- Can write good vs bad acceptance criteria and explain the difference


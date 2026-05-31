# Day 01 — QA Foundations: Mindset, SDLC, STLC
**Date:** 2026-05-18
**Phase:** 1 — Foundations
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com

---

## 1. QA vs QC vs Testing

**QA (Quality Assurance)**
Process-oriented. Prevents defects from being introduced. Involved from Day 1 of the project — in requirement reviews, sprint planning, and process improvement. Goal: build quality in, not test quality in.

**QC (Quality Control)**
Product-oriented. Checks the built software against requirements. Happens after something is built. Think of it as inspection.

**Testing**
The actual execution — running test cases, interacting with the app, running scripts. Testing is a tool used by both QA and QC.

**Real-world analogy:**
In a restaurant — QA is training the chefs and designing the kitchen process. QC is the head chef tasting the dish before it leaves the kitchen. Testing is the act of tasting it.

---

## 2. SDLC — Software Development Life Cycle

| Phase | What happens |
|-------|-------------|
| 1. Requirements | Business defines what they need |
| 2. System Design | Architects decide how to build it |
| 3. Development | Developers write the code |
| 4. Testing | QA verifies it works correctly |
| 5. Deployment | Code goes live to users |
| 6. Maintenance | Bug fixes and updates post-launch |

**Key insight:** In Agile teams, QA is involved from Phase 1. This is Shift-Left Testing. A bug found in requirements costs almost nothing. The same bug in production costs 100x more.

---

## 3. STLC — Software Testing Life Cycle

| Phase | Activities | Output Document |
|-------|-----------|-----------------|
| Requirement Analysis | Review requirements, identify test scenarios, flag gaps | RTM (Requirement Traceability Matrix) |
| Test Planning | Define scope, approach, schedule, risks, entry/exit criteria | Test Plan |
| Test Case Design | Write detailed test cases and scripts | Test Cases, Test Scripts |
| Test Environment Setup | Set up servers, databases, test accounts | Environment Checklist |
| Test Execution | Run tests, log bugs, retest fixes | Bug Reports, Execution Report |
| Test Closure | Summarise results, lessons learned | Test Summary Report |

---

## 4. The 7 Principles of Software Testing

| # | Principle | Meaning |
|---|-----------|---------|
| 1 | Testing shows presence of defects | Zero bugs found ≠ zero bugs exist |
| 2 | Exhaustive testing is impossible | Prioritise by risk |
| 3 | Early testing saves money | Bug in requirements = cheap. Bug in production = expensive |
| 4 | Defect clustering | 80% of bugs live in 20% of features |
| 5 | Pesticide paradox | Same tests stop finding new bugs — update them regularly |
| 6 | Testing is context dependent | Banking app ≠ mobile game testing approach |
| 7 | Absence of errors fallacy | Bug-free app that doesn't meet user needs is still a failure |

---

## 5. Key Concepts

**Severity vs Priority**
- Severity = technical impact (how badly the bug breaks the system)
- Priority = business urgency (how soon it must be fixed)
- Example: broken logo on an internal page = Low Severity, High Priority if launch is tomorrow

**Verification vs Validation**
- Verification: Are we building the product right? (reviews, walkthroughs)
- Validation: Are we building the right product? (actual testing)

**Test Oracle**
The source of truth used to determine if a test passed or failed — requirements doc, design spec, or common knowledge.

**Test Prioritization**
Which tests to run first — based on business risk, recent changes, areas with historical defects, and frequency of use.

---

## 6. OrangeHRM — Application Explored Today

OrangeHRM is a Human Resource Management System used by companies to manage employees, leave, payroll and recruitment.

**Modules found:**
- Login / Authentication
- Dashboard
- PIM — Personnel Information Management (employee records)
- Leave — apply, approve, reject leave requests
- Recruitment — job postings and candidate tracking
- Admin — user management and system configuration
- My Info — employee self-service

**User types:**
- Admin — full access to all modules
- ESS (Employee Self Service) — limited to own profile and leave

---

## Tools Set Up Today

| Tool | Purpose | URL |
|------|---------|-----|
| GitHub | Portfolio and version control | github.com |
| JIRA | Bug tracking | atlassian.com |
| TestRail | Test case management | testrail.com |
| Chrome DevTools | Network analysis | Built into Chrome (F12) |

---

## Interview Talking Points

- Can explain QA vs QC vs Testing with a real-world analogy
- Know all 6 STLC phases and their output documents
- Understand severity vs priority with examples
- Can explain shift-left testing and why it saves cost
- Hands-on experience writing and executing manual test cases on a real HR application
- Logged 5 formal bug reports in JIRA including a security vulnerability


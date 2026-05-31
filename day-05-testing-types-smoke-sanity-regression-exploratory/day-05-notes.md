# Day 05 — Testing Types: All Types Deep Dive
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com

---

## 1. Smoke Testing

Quick, high-level check on a new build to determine whether it is stable enough to proceed with detailed testing. Covers critical functionality only — happy paths of the most important features.

**Trigger:** New build received from development
**Who:** QA engineer
**Time:** 15–30 minutes maximum
**Covers:** 10–15% of total test suite — critical paths only
**If it fails:** Build rejected immediately. Sent back to development. No further testing.

**OrangeHRM smoke test:**
- App accessible at URL
- Admin can login
- New employee can be created
- Leave request can be submitted
- Dashboard loads without errors
- Logout works

**Fintech context (Barq):**
- Users can login to wallet
- Money transfer can be initiated
- Transaction appears in history
- API gateway responding
If any fail — production deployment stops.

---

## 2. Sanity Testing

Narrow, focused check after a bug fix to verify the specific fix works and has not broken anything immediately adjacent.

**Trigger:** Bug fix received from developer
**Scope:** Specific area that was fixed + immediate neighbours
**Time:** 5–15 minutes

### Smoke vs Sanity — the most confused pair

| | Smoke | Sanity |
|--|-------|--------|
| Trigger | New build | Bug fix |
| Scope | Whole app — critical paths | Specific fixed area only |
| Depth | Shallow | Moderate |
| Goal | Is build stable? | Is this fix working? |
| Time | 15–30 mins | 5–15 mins |

**Example:** After BUG-007 fix (leave approval with zero balance):
- Try approving with zero balance → should be blocked ✅
- Try approving with positive balance → should still work ✅
- Check balance updates correctly after approval ✅
Only these 3 scenarios. Not the full leave module.

---

## 3. Regression Testing

Re-executing a predefined set of test cases on a new build to verify previously working functionality has not been broken by new code changes.

**Trigger:** Every sprint before release, after any significant change, before every production deployment
**Purpose:** Protect existing features — not test new ones

### Types of Regression

**Full regression:** Entire test suite executed. Before major releases. Usually automated.

**Partial/Selective regression:** Only test cases related to changed areas. For minor releases or patches.

**Progressive regression:** New test cases added as new features are validated.

### Regression test selection criteria
- Most-used features (high business impact)
- Integration points between modules
- Test cases that historically found bugs
- Recently changed areas
- All smoke test cases (always included)

---

## 4. Retesting

Re-executing the exact test case that found a specific bug — after the developer fixes it — to verify the fix works.

### Retesting vs Regression

| | Retesting | Regression |
|--|-----------|-----------|
| Purpose | Verify specific bug fixed | Verify nothing else broke |
| Scope | Exact failing test case only | Broad set of test cases |
| Trigger | Developer marks bug Fixed | New build or release |

**Retesting checklist:**
- Use exact same test data as original bug
- Same environment and browser
- Follow exact steps from bug report
- Document result — pass or still failing
- If still failing — reopen with comment explaining what was found

---

## 5. Exploratory Testing

Simultaneous learning, test design, and test execution. No pre-written test cases. The tester explores using knowledge, intuition, and curiosity.

**When to use:**
- Incomplete or changing requirements
- New features never tested before
- After major refactor
- When scripted tests all pass but something feels wrong

### Session-Based Exploratory Testing (SBET)

Structured exploration using a written charter:
- **Goal:** What area are you exploring?
- **Scope:** Which features or scenarios?
- **Time-box:** 45–90 minutes
- **Notes:** Document findings as you go

**Example charter:**
```
Goal: Explore leave cancellation edge cases
Scope: Approved leave starting today, leave with public holidays,
       leave cancelled on last day
Time-box: 60 minutes
```

### SFDIPOT Heuristic

| Letter | Meaning | Questions to ask |
|--------|---------|-----------------|
| S | Structure | What is the app made of? Test each element |
| F | Function | What should each function do? What should it NOT do? |
| D | Data | Valid/invalid/boundary/special characters |
| I | Integration | How does it connect to other systems? What breaks when they fail? |
| P | Platform | Which browsers, OS, devices, screen sizes? |
| O | Operations | What happens after long use? Many users? After restart? |
| T | Time | What happens at midnight? Month-end? After daylight saving change? |

---

## 6. Ad-hoc Testing

Completely unplanned, undocumented testing with no structure. No charter, no session, no documentation.

**When useful:** Quick informal check, developer self-testing, pair testing
**Not a substitute for exploratory testing** — cannot be repeated or measured

### Exploratory vs Ad-hoc

| | Exploratory | Ad-hoc |
|--|-------------|--------|
| Structure | Charter-based, time-boxed | No structure |
| Documentation | Notes during session | None |
| Repeatability | Can be repeated | Cannot |
| Professionalism | Recognised QA technique | Informal only |

---

## 7. Monkey Testing

Random inputs provided without defined test cases or expected outcomes. Goal: crash the system or find unhandled exceptions.

| Type | Description |
|------|-------------|
| Dumb Monkey | Completely random — no app knowledge |
| Smart Monkey | Random within valid app boundaries |
| Brilliant Monkey | Random but uses domain knowledge |

**Tools:** Android Monkey (built into Android SDK), Gremlins.js (web)
**Best for:** Stress testing stability, finding unhandled exceptions

---

## 8. Gorilla Testing

Exhaustively testing one specific module to its absolute limit — every possible variation, edge case, and scenario.

**When to use:**
- Critical feature that cannot fail (payments, authentication)
- Module with high historical bug density
- Newly built module never tested before

**Example — gorilla testing OrangeHRM login:**
Every browser, every device, slow network, JavaScript disabled, password manager, browser zoom 200%, keyboard-only navigation, after cookie deletion, concurrent logins, after session expiry.

### Monkey vs Gorilla

| | Monkey | Gorilla |
|--|--------|---------|
| Coverage | Entire app, random | One feature, exhaustive |
| Approach | Random, unplanned | Systematic, thorough |
| Goal | Find crashes anywhere | Validate one area completely |

---

## 9. UAT — User Acceptance Testing

Testing performed by the actual business users or clients — not QA — to verify the software meets their business requirements and is ready for production use.

**Who does it:** Business users, product owners, clients
**When:** After QA sign-off, before production release
**Environment:** UAT environment — as close to production as possible
**Goal:** "Does this software do what the business needs it to do?"

**Types of UAT:**
- **Alpha testing:** Done by internal users at the developer's site
- **Beta testing:** Done by external users in their own environment — real-world conditions

### Alpha vs Beta

| | Alpha | Beta |
|--|-------|------|
| Who | Internal employees | External real users |
| Location | Developer's environment | User's own environment |
| Control | High — monitored closely | Low — users test freely |
| Purpose | Find bugs before public | Real-world feedback before launch |

---

## 10. A/B Testing

Two versions of a feature shown to different user segments simultaneously. Behaviour measured to determine which version performs better.

**QA's role in A/B testing:**
- Validate both Version A and Version B work correctly
- Verify users are consistently shown the same version (not switching between sessions)
- Verify metrics are being captured correctly
- Verify experiment is set up for the right user percentage

---

## 11. Shadow Testing

Real production traffic is mirrored to the new system in parallel. New system's responses compared to production responses. Users only see production response — shadow system used for comparison only.

**Use case:** Validating a new backend service before switching traffic to it. Zero user risk.

---

## 12. Canary Testing and Blue-Green Deployment

**Canary:** Deploy new version to 5% of users. Monitor error rates and performance. If healthy — expand to 10%, 25%, 50%, 100%. If metrics spike — rollback immediately.

**Blue-Green:** Two identical production environments. Blue = live, Green = new version. Traffic switches from Blue to Green instantly. If anything goes wrong — switch back in seconds.

---

## 13. Master Summary Table — All Testing Types

| Type | Trigger | Scope | Depth | Goal |
|------|---------|-------|-------|------|
| Smoke | New build | Whole app | Shallow | Is build stable? |
| Sanity | Bug fix | Fixed area only | Moderate | Is fix working? |
| Regression | New build/release | Previously tested areas | Full | Did new code break old features? |
| Retesting | Bug marked Fixed | Exact failing test | Exact | Is specific bug fixed? |
| Exploratory | New feature / intuition | Chosen scope | Deep, creative | Find unknown bugs |
| Ad-hoc | Anytime | Random | Unstructured | Quick informal check |
| Monkey | Stability testing | Whole app | Random | Can random usage crash it? |
| Gorilla | Critical feature | One feature | Exhaustive | Is this feature bulletproof? |
| UAT | After QA sign-off | Business flows | Business-focused | Does it meet user needs? |
| Alpha | Pre-release | Full app | Internal | Internal validation |
| Beta | Pre-release | Full app | Real-world | External real-user feedback |
| A/B | Live experiment | Specific feature | Comparative | Which version performs better? |
| Shadow | New system validation | Specific service | Comparison | Does new system behave like old? |
| Canary | Deployment | % of users | Monitored | Safe gradual rollout |

---

## Interview Talking Points

- Can precisely define every testing type with trigger, scope, and goal
- Know smoke vs sanity — the most commonly confused pair in interviews
- Know exactly when to run regression vs retesting and why they differ
- Can explain session-based exploratory testing with SBET charter example
- Know the SFDIPOT heuristic and can apply it to any feature
- Can explain monkey vs gorilla with real use cases
- Can explain all four deployment testing strategies — canary, blue-green, shadow, A/B


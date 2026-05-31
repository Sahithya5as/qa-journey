# Day 03 — Bug Lifecycle, Defect Metrics & Advanced Bug Terminology
**Phase:** 1 — Manual Testing Deep Dive
**Environment:** macOS | Chrome
**App Under Test:** OrangeHRM — https://opensource-demo.orangehrmlive.com

---

## 1. Bug Lifecycle — Every State Explained

A bug is not just "open" or "closed." It passes through a defined set of states. Understanding the full lifecycle is what separates a junior tester from a senior QA engineer. Every state has a reason, a person responsible, and an action required.

```
New → Assigned → Open → Fixed → Retest → Closed
                              ↘ Reopen (if fix failed)
         ↘ Rejected (not a bug)
         ↘ Deferred (valid but not now)
         ↘ Duplicate (already reported)
         ↘ Cannot Reproduce
```

### State-by-state breakdown

**New**
The bug has just been raised by the QA engineer. It has not been reviewed by anyone yet. At this point the bug report should be complete — steps, screenshots, severity, priority all filled in. An incomplete bug report at this stage wastes the developer's time.

**Assigned**
The bug has been reviewed by the QA lead or project manager and assigned to a specific developer. If the bug is unclear, it may be sent back to QA for more information before assignment.

**Open (In Progress)**
The developer has picked up the bug and is actively working on a fix. The status changes from Assigned to Open when the developer starts working on it.

**Fixed**
The developer has made a code change they believe resolves the bug. The bug is handed back to QA for verification. At this point the developer should write what they changed so the QA engineer knows what to focus retesting on.

**Retest**
QA receives the fixed build and re-executes the exact steps from the original bug report. This is called retesting. QA does not just check the fixed scenario — they also run regression tests on related areas to make sure the fix did not break something else.

**Closed**
The bug is verified as fixed by QA. The fix works, regression is clean. Bug is closed.

**Reopened**
QA retested and the bug still exists, or the fix caused a new bug in the same area. The bug goes back to Open status with a comment explaining what was found during retest.

**Rejected**
The developer or BA reviews the bug and determines it is not actually a bug — either it is working as designed, or the requirement is ambiguous. QA should challenge this with evidence — the requirement, the design spec, or user expectations. If QA agrees it is working as designed, the test case should be updated to reflect the correct behavior.

**Deferred**
The bug is valid — it is a real defect — but the business has decided not to fix it in the current release. Common reasons: low priority, not enough time, complex fix needed for a minor issue. Deferred bugs must be tracked and revisited in future sprints.

**Duplicate**
The same bug has already been reported by someone else. The duplicate is linked to the original and closed. The original bug remains open.

**Cannot Reproduce**
The developer could not reproduce the bug using the steps provided. This is a common frustration. When this happens, QA should: verify the steps are complete and exact, check if the issue is environment-specific, provide a screen recording, and check if it is intermittent. Never accept "cannot reproduce" without investigation.

---

## 2. Severity vs Priority — Deep Dive

These are the two most asked about concepts in every QA interview. Most people give a surface answer. Here is the complete picture.

### Severity
Defined by the QA engineer. Measures the technical impact on the system.

| Severity | Definition | Example |
|----------|-----------|---------|
| Critical (S1) | System crash, data loss, security breach, complete feature failure | App crashes on login, database corrupted, SQL injection succeeds |
| High (S2) | Major feature broken, no workaround available | Cannot create a new employee, cannot submit leave request |
| Medium (S3) | Feature works but behaves incorrectly, workaround exists | Wrong employee count shown but data is correct, can refresh to fix |
| Low (S4) | Cosmetic issue, minor UX problem, no functional impact | Button text misaligned, wrong font colour, typo in label |

### Priority
Defined by the Product Manager or Business. Measures how urgently the bug needs to be fixed.

| Priority | Definition | Example |
|----------|-----------|---------|
| P1 — Critical | Fix today, blocks everything, production is down | Payment gateway is down on a live e-commerce site |
| P2 — High | Fix this sprint, impacts users significantly | Core feature broken in current release |
| P3 — Medium | Fix next sprint, workaround exists | Minor feature behaving incorrectly, low user impact |
| P4 — Low | Fix when time allows, cosmetic or trivial | Typo in footer text |

### The 4 combinations every interviewer asks about

**High Severity + High Priority**
App crashes on the checkout page of a live e-commerce site. Fix immediately.

**High Severity + Low Priority**
A critical crash happens in a feature used by 0.1% of users on an old browser version being deprecated next month. Real defect, but not urgent.

**Low Severity + High Priority**
The company logo is broken on the homepage one hour before a major product launch event. Cosmetic only, but must be fixed right now.

**Low Severity + Low Priority**
A typo in the help documentation of a rarely-used admin panel. Real issue, no rush.

---

## 3. Advanced Bug Terminology

### Bug Leakage
A bug that was present during testing but was not found by QA — and was later discovered by the end user in production. This is a QA failure. Bug leakage is measured as a KPI.

**Formula:** Bug Leakage % = (Bugs found in production ÷ Total bugs found) × 100

**Example:** QA found 45 bugs during testing. After release, 5 bugs were reported by users. Bug leakage = (5 ÷ 50) × 100 = 10%. A good team targets below 5%.

**Why it happens:**
- Insufficient test coverage
- Requirements not reviewed properly
- Edge cases missed
- Not enough regression testing after fixes
- Time pressure leading to skipped test areas

---

### Bug Density
Measures the number of bugs found per unit of software — usually per thousand lines of code (KLOC) or per feature/module.

**Formula:** Bug Density = Total bugs found ÷ Size of software (in KLOC or modules)

**Why it matters:** Bug density helps identify which modules are the most defect-prone. If the Leave module consistently has high bug density across releases, that tells QA to focus more testing effort there. It also helps predict where bugs are likely to hide in new features built on the same code.

**In practice:** As a QA engineer, you track bug density per module in your test summary report. "The Leave module had a bug density of 4.2 per KLOC this sprint — highest across all modules. Recommending additional unit test coverage by development."

---

### Bug Bash
A time-boxed, all-hands testing event where everyone — developers, QA, product managers, designers, even non-technical stakeholders — tests the application simultaneously for a fixed period (usually 2–4 hours). The goal is to find as many bugs as possible before a release.

**Why it works:**
- Different people use the app in completely different ways
- Non-technical users find UX issues that QA misses
- Developers often find bugs in their own code when they use it as a user
- Creates shared ownership of quality

**QA's role in a bug bash:**
- Set up the environment before the session
- Brief all participants on how to log bugs (template, severity guide)
- Coordinate so people test different areas
- Review and triage all bugs raised after the session

---

### Bug Triage
A meeting where the team reviews newly raised bugs, validates them, sets severity and priority, assigns them, and decides what gets fixed in the current sprint. Attended by QA, development lead, and product manager.

**QA's role in triage:**
- Present evidence for each bug (steps, screenshots, video)
- Defend severity ratings with data, not opinion
- Challenge rejected bugs with requirement references
- Accept deferral when business priority is clear

---

### Showstopper Bug
A bug so severe it blocks the release entirely. All testing stops until this is fixed and retested. Criteria: the main user journey is completely broken, data loss risk exists, or a security vulnerability is exposed.

---

### Flaky Bug (Intermittent Defect)
A bug that only reproduces sometimes — not consistently. These are the hardest to fix and the most frustrating. QA's job when finding a flaky bug:
- Try to reproduce it at least 3 times
- Document every condition when it did reproduce (browser, network speed, user actions before)
- Try to find a pattern (only on slow networks? only after login? only on certain data?)
- Record a video — "cannot reproduce" is much harder to claim when there is a video

---

### Regression Bug
A bug that was previously fixed but has reappeared in a new release — because a new code change broke the old fix. This is why regression testing exists. Every time code is released, QA runs regression tests on previously fixed bugs to confirm they have not come back.

---

### Zero-Day Bug
A security vulnerability that is unknown to the software vendor and has not been patched. In a QA context, this is a security defect found during security testing that the development team was not aware of. Requires immediate escalation.

---

## 4. Bug Report Best Practices — Senior Level

A senior QA engineer's bug reports are accepted immediately with no back-and-forth. Here is what makes them different:

**Title formula:**
`[Module] — [What failed] — [Condition when it fails] — [Environment]`

**Good examples:**
- `Leave Module — Manager can approve leave with zero employee balance — no warning shown — Chrome v124 / macOS`
- `Login — Account not locked after 5 consecutive failed password attempts — Security vulnerability — All browsers`
- `PIM — Employee profile photo upload accepts executable files (.exe) — File type validation missing — Chrome v124`

**What separates a senior bug report:**
- Steps are so precise a developer who has never seen the app can reproduce it in under 2 minutes
- Expected result references the requirement or design spec, not just common sense
- Actual result includes the exact error message, exact data, exact HTTP status code if relevant
- Attachments always included — screenshot minimum, screen recording for intermittent bugs
- Environment section includes app version, not just browser and OS
- Impact section explains the business consequence, not just the technical symptom

---

## 5. Bug Metrics — What QA Reports to Management

Every sprint, QA produces a metrics report. These are the numbers that matter:

| Metric | Formula | What it tells you |
|--------|---------|-------------------|
| Defect Detection Rate | Bugs found by QA ÷ Total bugs × 100 | How effective QA is at catching bugs before release |
| Defect Leakage Rate | Bugs found in production ÷ Total bugs × 100 | How many bugs escaped QA |
| Bug Density | Total bugs ÷ Module size (KLOC) | Which modules are most defect-prone |
| Defect Removal Efficiency (DRE) | Bugs found before release ÷ Total bugs × 100 | Overall team quality |
| Reopen Rate | Reopened bugs ÷ Total fixed bugs × 100 | How well developers are fixing bugs |
| Mean Time to Detect (MTTD) | Average time from bug introduction to discovery | How quickly bugs are being caught |
| Mean Time to Resolve (MTTR) | Average time from bug report to fix verified | How fast the team resolves defects |

**In an interview:**
"In my last project, our defect leakage rate was 8% in Q1. I introduced exploratory testing sessions and improved our regression suite coverage. By Q3 we had reduced leakage to 3%."

That is what 9 years of QA experience sounds like.

---

## 6. Bug Bash — How to Run One (Practical Guide)

**Before the session (QA's responsibility):**
- Prepare the test environment — stable build, test data ready
- Create a bug logging template everyone will use
- Assign areas to participants to avoid duplication
- Set up a shared screen or Confluence page for real-time logging

**During the session:**
- Set a timer — 2 to 4 hours maximum
- Everyone tests simultaneously
- Log bugs as you find them — do not wait until the end
- Focus on user journeys, not isolated fields

**After the session:**
- QA triages all bugs raised — remove duplicates, validate severity
- Group by module — identify where most bugs clustered
- Prioritise with product manager
- Add valid bugs to the sprint backlog

---

## Interview Talking Points from Today

- Can walk through every bug lifecycle state and explain who is responsible at each stage
- Know the difference between a Rejected bug and a Deferred bug and when to fight back
- Can calculate and explain bug leakage, bug density, DRE with formulas
- Know the difference between flaky, regression, and showstopper bugs
- Have run bug bash sessions and can describe the process end to end
- Can present bug metrics to management: leakage rate, density, MTTR
- Write senior-level bug reports that get accepted immediately with no back-and-forth


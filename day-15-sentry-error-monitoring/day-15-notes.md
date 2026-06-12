# Day 15 — Sentry Error Monitoring
**Phase:** 2 — Network, Logs & Monitoring
**Tool:** Sentry (sentry.io) — free account
**Project:** qa-payment-investigation
**Environment:** staging | Release: payment-app@1.0.0

---

## 1. What is Sentry and Why It Matters

Yesterday you read log files manually using grep. That works for small systems. But a system processing 1 million requests per day generates millions of log lines — impossible to grep manually.

Sentry solves this by:
- Automatically capturing every error in real time
- Grouping similar errors together so you see patterns not noise
- Showing exactly which line of code caused the crash
- Telling you how many users were affected
- Alerting you the moment a new error appears
- Tracking which release introduced the error
- Showing the user's browser, OS, and session context automatically

**Think of Sentry as:** grep running 24/7 on steroids, with automatic grouping, user impact tracking, and instant alerts.

**Why QA managers need Sentry:**
After every release you watch Sentry for 30 minutes. If error rate spikes — you catch it before customers complain. This is shift-right testing in practice.

---

## 2. Sentry vs Firebase Crashlytics

| Feature | Sentry | Firebase Crashlytics |
|---------|--------|---------------------|
| Mobile crashes | ✅ Yes | ✅ Yes |
| Web app errors | ✅ Yes | ❌ No |
| Backend API errors | ✅ Yes | ❌ No |
| Performance monitoring | ✅ Yes | Limited |
| Release tracking | ✅ Yes | Limited |
| User context | ✅ Yes | Limited |
| Free tier | ✅ Yes | ✅ Yes |

---

## 3. Sentry Dashboard — What Each Section Means

### Issues List

Each row shows one grouped error:
```
Issue title          Last Seen   Age    Events   Users   Priority
TypeError: null...   1min ago    1min   2        1       Medium
ReferenceError...    1min ago    1min   1        1       High
```

| Column | Meaning |
|--------|---------|
| Issue | Error type + message + source file |
| Last Seen | When did this error most recently fire |
| Age | How long since first occurrence |
| Events | How many times this error has fired total |
| Users | How many unique users experienced it |
| Unhandled badge | Orange — no try/catch — app crashed with no recovery |

### Issue Detail Page

Each section tells you something specific:

**Events and Users cards:**
Events = total times this error fired. Users = unique people affected. Events > Users means the same users hitting the error repeatedly.

**Browser/Release/Environment bar:**
Shows distribution across browser versions, releases, and environments. 100% Chrome 149.0.0 means this only happens on that specific browser version.

**User context row:**
```
testqa@payment.com  USR001  Chrome 149.0.0  Mac OS X >=10.15.7  1.0.0  staging
```
Captured automatically from `Sentry.setUser()`. In production this shows exactly which customer was affected — enabling proactive support contact.

**Highlights section:**
```
handled:     yes/no     ← was there a try/catch?
level:       error      ← severity level
transaction: --         ← which API endpoint
```

**Stack Trace:**
The exact code path that led to the crash. Read from top — the top line is where it crashed, lines below show what called it.

---

## 4. The Most Important Concept — Handled vs Unhandled

### Handled Error (handled: yes)
The developer wrote a try/catch block. The code anticipated that something could go wrong and planned for it.

```javascript
// Handled — developer expected this could fail
try {
    payment.processTransaction();
} catch(e) {
    Sentry.captureException(e);  // logged to Sentry
    showUserMessage("Payment failed, please try again");  // user sees friendly message
    // app continues working
}
```

**What happens:** Code fails → try/catch catches it → Sentry logs it → user sees friendly message → app continues running.

### Unhandled Error (handled: no)
No try/catch. The code had no plan for this failure.

```javascript
// Unhandled — no safety net
undefinedPaymentFunction();  // this function does not exist
// app crashes completely
// Sentry catches it via global window.onerror handler
```

**What happens:** Code fails → nothing catches it → app breaks completely → user sees blank screen or frozen UI → Sentry's global error handler catches it automatically.

### Why Unhandled errors are more dangerous

| | Handled | Unhandled |
|--|---------|-----------|
| App state after error | Still running | Crashed/broken |
| User experience | Friendly error message | Blank screen or freeze |
| Recovery possible | Yes | No |
| Caught by | Developer's try/catch | Sentry's global handler |
| In Sentry | No orange badge | 🟠 Unhandled badge |

**Interview answer:** "I prioritise Unhandled errors over Handled ones because Unhandled errors mean the app has completely crashed with no recovery path — the user is stuck. Handled errors mean the developer anticipated the failure and the app can continue, so the impact is lower even if the error rate is similar."

---

## 5. captureException vs captureMessage

### captureException
Sends an actual error object to Sentry. Includes a full stack trace showing exactly which line of code caused the problem.

```javascript
try {
    payment.processTransaction();
} catch(e) {
    Sentry.captureException(e);  // sends error WITH stack trace
}
```

Use when: something broke — a crash, unexpected failure, exception was thrown.

### captureMessage
Sends a custom text message to Sentry WITHOUT a stack trace. Used when something important happened that is NOT a crash but needs monitoring.

```javascript
// Duplicate transaction detected — app did not crash but this is serious
Sentry.captureMessage("Duplicate transaction detected: TXN003 processed twice");
```

Use when: a business rule was violated, a suspicious pattern was detected, or an important event should be tracked — but no exception was thrown.

**Real-world examples of captureMessage:**
- Duplicate transaction detected
- Suspicious login attempt from new country
- Payment amount exceeded daily limit
- User attempted to access admin without permission

---

## 6. Real Errors Found and Analysed Today

### Error 1 — TypeError: Cannot read properties of null (JAVASCRIPT-1)
```
handled: yes ← developer wrote try/catch
mechanism: generic
Events: 2 | Users: 1
Stack trace: TypeError — Cannot read properties of null (reading 'processTransaction')
Triggered by: triggerNullPayment()
```
**Analysis:** Payment object was null — developer correctly caught this with try/catch. App did not crash. This represents what happens when a payment is initiated before the payment object is initialised.

### Error 2 — Error: Insufficient balance (JAVASCRIPT-2)
```
handled: yes
level: warning ← lower severity, set explicitly
Events: 1 | Users: 1
Extra data: required_amount=500, available_balance=200, txn_id=TXN002
```
**Analysis:** User attempted to pay ₹500 with only ₹200 balance. Correctly handled — not a crash. The extra data (required vs available amount) is exactly what support teams need to resolve customer complaints.

### Error 3 — Error: Database connection pool exhausted (JAVASCRIPT-3)
```
handled: yes
level: fatal ← highest severity, set explicitly
Events: 1 | Users: 1
Extra data: max_connections=10, active_connections=10, waiting=5
```
**Analysis:** Same database outage found in log analysis yesterday — now confirmed in Sentry with full context. The `fatal` level correctly signals this is a system-wide issue.

### Error 4 — Message: Duplicate transaction TXN003 (JAVASCRIPT-4)
```
handled: -- ← captureMessage, not an exception
level: error
Message: "Duplicate transaction detected: TXN003 processed twice"
Extra data: original_time=08:10:05, duplicate_time=08:10:05
```
**Analysis:** This is a business logic violation captured as a message — no stack trace because no exception was thrown. The duplicate charge bug from Day 14 confirmed in Sentry.

### Error 5 — ReferenceError: undefinedPaymentFunction (JAVASCRIPT-5)
```
handled: no ← UNHANDLED
mechanism: onerror ← caught by global browser handler
Events: 1 | Users: 1
Stack trace: ReferenceError — undefinedPaymentFunction is not defined
```
**Analysis:** Most dangerous error. Called a function that does not exist — complete app crash. No try/catch. Sentry's global `onerror` handler saved it. In production this leaves users completely stuck.

### Error 6 — TypeError: Cannot read properties of undefined — setData (JAVASCRIPT-6)
```
handled: no ← UNHANDLED
mechanism: setTimeout + instrument ← happened inside a timer
Events: 2 | Users: 1
Stack trace: TypeError — Cannot read properties of undefined (reading 'setData')
```
**Analysis:** The slow transaction simulation — after a 3400ms delay the Sentry transaction object was undefined when `.setData()` was called. Unhandled because it happened inside a setTimeout where the outer try/catch could not reach it.

---

## 7. Priority Framework — How to Triage Sentry Issues as QA Manager

### Step 1 — Separate Unhandled from Handled
Unhandled = app crashes = highest priority regardless of frequency.

### Step 2 — Assess business impact
- Financial impact (duplicate charge, payment failure) → P1
- System down (database exhausted) → P1
- App crash (unhandled errors) → P1-P2 based on user count
- Functional errors (insufficient balance) → P2-P3

### Step 3 — Check user count and trend
- Many users + increasing trend = escalate immediately
- Few users + flat trend = schedule for next sprint

### Priority applied to today's findings:

| Priority | Issue | Reason |
|----------|-------|--------|
| P1 | Duplicate transaction TXN003 | Direct financial impact — user charged twice |
| P1 | Database connection pool exhausted | System-wide outage — all users affected |
| P1 | ReferenceError — Unhandled | App crash — no recovery — any user hitting this is stuck |
| P2 | TypeError setData — Unhandled | App crash in specific flow — needs fix before next release |
| P3 | Insufficient balance error | Expected behaviour — handled correctly |
| P3 | TypeError processTransaction | Handled correctly — friendly message shown |

---

## 8. Sentry for QA Manager — Release Monitoring

After every deployment:
1. Go to Sentry Issues — filter by `First Seen: last 1 hour`
2. Check for any new Unhandled errors — these are regressions
3. Check error rate trend — spike after deploy = regression
4. Check crash-free users % — should be above 99.5%
5. Compare with previous release — `Issues → Releases tab`

**Interview answer for "how do you monitor quality after a release?"**
*"I watch Sentry for the first 30 minutes after every deployment. I filter Issues by 'First Seen: last hour' to see any new errors introduced by this release. I also check the crash-free users percentage — if it drops below 99.5% I immediately raise an incident. I use the Releases comparison to diff error rates between the new and previous release. If I see any new Unhandled errors I recommend a rollback until the root cause is identified."*

---

## Interview Talking Points

- Can explain what Sentry does and why it is better than manual log analysis at scale
- Know the difference between Handled and Unhandled errors and why Unhandled is more critical
- Know the difference between captureException (with stack trace) and captureMessage (without)
- Can read a Sentry issue detail page and extract root cause, user impact, and browser context
- Can prioritise 6 issues by business impact — financial, system-wide, app crash, functional
- Know how to use Sentry for post-release monitoring as a QA manager
- Found and documented the duplicate transaction, database outage, and two Unhandled crashes


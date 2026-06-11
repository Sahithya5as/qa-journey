# Day 14 — Bug Reports: Backend Log Analysis Findings
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS Terminal — log analysis
**Source:** app.log — payment service simulation

---

## Summary

| Bug ID | Title | Severity | Priority | Found By |
|--------|-------|----------|----------|----------|
| BUG-L001 | Duplicate transaction processed — user charged twice — no idempotency | Critical | P1 | Log analysis — grep TXN003 |
| BUG-L002 | Duplicate transaction detection delay 35 minutes — should be under 60 seconds | High | P1 | Log timestamp comparison |
| BUG-L003 | Account lockout threshold 5 attempts — too permissive for fintech | Medium | P2 | Log analysis — brute force pattern |
| BUG-L004 | Slow database query 3400ms — missing index on user_id | High | P2 | grep slow query |
| BUG-L005 | Database connection pool has no early warning monitoring | High | P1 | Log analysis — pool exhausted before FATAL |

---

## BUG-L001 — CRITICAL

**Title:** Payment Service — Duplicate transaction TXN003 processed twice — user USR003 charged ₹250 twice — idempotency not implemented

**Severity:** Critical
**Priority:** P1
**Type:** Data Integrity — Financial
**Found by:** Log analysis — `grep "TXN003" app.log`

**Log Evidence:**
```
08:10:05 INFO [payment-service] Payment initiated txn_id=TXN003 amount=250 user_id=USR003
08:10:05 INFO [payment-service] Payment initiated txn_id=TXN003 amount=250 user_id=USR003  ← DUPLICATE
08:10:07 INFO [payment-service] Payment successful txn_id=TXN003 duration_ms=456
08:10:07 INFO [payment-service] Payment successful txn_id=TXN003 duration_ms=456  ← DUPLICATE
08:45:00 ERROR [payment-service] Duplicate transaction detected txn_id=TXN003
```

**Root Cause:**
No idempotency key implemented on the payment initiation endpoint. The client sent two identical requests at the same timestamp — likely caused by double-tap, button not disabled after first click, or network retry logic. The server processed both requests independently instead of recognising them as the same operation.

**Business Impact:**
User USR003 was charged ₹250 twice. Requires manual refund investigation. In a high-volume payment system this could affect thousands of transactions daily. Financial regulatory bodies (RBI, SAMA) require financial systems to guarantee exactly-once payment processing.

**Recommendation:**
1. Implement idempotency keys — client generates a unique key per payment attempt, server rejects duplicate keys
2. Add server-side duplicate detection with a 60-second window
3. Disable payment button on client after first tap — re-enable only on failure

---

## BUG-L002 — HIGH

**Title:** Payment Service — Duplicate transaction detection delayed 35 minutes — real-time detection required for payment systems

**Severity:** High
**Priority:** P1
**Type:** Monitoring Gap
**Found by:** Log timestamp comparison

**Evidence:**
```
08:10:05 — Duplicate transaction occurred
08:45:00 — Duplicate transaction detected (35 minutes later)
Gap: 34 minutes 55 seconds
```

**Expected:** Duplicate transaction detected within 60 seconds — alert sent to operations team immediately.
**Actual:** 35-minute detection gap. User would have already received two debit notifications before detection.

**Impact:** 35 minutes is enough time for a user to contact their bank, dispute the charge, and cause a chargeback. Early detection enables proactive customer communication before the user notices.

---

## BUG-L003 — MEDIUM

**Title:** Auth Service — Account lockout threshold set to 5 failed attempts — should be 3 for a payment application

**Severity:** Medium
**Priority:** P2
**Type:** Security Configuration
**Found by:** Log analysis — brute force pattern

**Evidence:**
```
08:01:15 WARN attempts=1
08:01:16 WARN attempts=2
08:01:17 WARN attempts=3
08:01:18 WARN attempts=4
08:01:19 ERROR Account locked attempts=5
```

**Expected:** For a payment application, account should lock after 3 failed attempts per RBI guidelines and general fintech security best practices.
**Actual:** Account locks after 5 attempts — giving attacker 2 extra guesses.

**Additional gap:** No IP-based rate limiting observed. Attacker from 192.168.1.99 could attack multiple different accounts — each getting 5 attempts before lockout.

**Recommendation:**
- Reduce lockout threshold to 3 for payment accounts
- Add IP-based rate limiting — block IP after 5 failed attempts across any accounts
- Add CAPTCHA after 2 failed attempts
- Alert security team on brute force pattern detection in real time

---

## BUG-L004 — HIGH

**Title:** Payment Service — Database query 3400ms on transactions table — missing index on user_id column causes full table scan

**Severity:** High
**Priority:** P2
**Type:** Performance — Database
**Found by:** `grep "Slow query" app.log`

**Evidence:**
```
08:25:00 WARN [payment-service] Slow query detected 
duration_ms=3400 
query="SELECT * FROM transactions WHERE user_id=?"
```

**Root Cause:** Missing database index on user_id column. Without an index, the database performs a full table scan — reading every row to find matching records. At scale with millions of transactions this becomes a 30+ second query.

**Business Impact:**
- Users wait 3.4 seconds to view their transaction history
- Under load with concurrent users, this query competes for database resources
- Could contribute to connection pool exhaustion (linked to BUG-L005)

**Recommendation:**
```sql
CREATE INDEX idx_transactions_user_id ON transactions(user_id);
-- Or composite if date filtering is common:
CREATE INDEX idx_transactions_user_date ON transactions(user_id, created_at DESC);
```
Expected improvement: 3400ms → under 50ms.

---

## BUG-L005 — HIGH

**Title:** Database — Connection pool exhaustion detected at 100% capacity — no early warning monitoring causing FATAL outage

**Severity:** High
**Priority:** P1
**Type:** Infrastructure — Monitoring Gap
**Found by:** Database incident timeline analysis

**Evidence:**
```
08:15:00 ERROR Connection pool exhausted max=10 active=10 waiting=5
08:15:03 FATAL Database connection lost ← 3 seconds later
Duration down: 5 minutes
```

**Root Cause:** No monitoring alert at 80% connection pool usage. The first indication of a problem was when the pool was already 100% full with 5 requests waiting. This escalated to a complete database failure within 3 seconds — no time to intervene.

**Business Impact:** 5 minutes of complete payment service downtime. All users attempting payments during 08:15–08:20 received failures. Estimated revenue loss depends on transaction volume.

**Recommendation:**
- Set Datadog/Grafana alert at 80% connection pool usage — gives time to scale or investigate
- Implement circuit breaker — stop sending requests to database when pool is near full
- Consider connection pool auto-scaling
- Add runbook for database connection exhaustion — on-call engineer knows exactly what to do


# Day 14 — Backend Server Log Analysis
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS — Terminal
**Tools:** grep, cat, wc, awk, uniq, sort
**Log file analysed:** app.log — payment service simulation

---

## 1. What Are Backend Server Logs

Every server running an API writes logs to files. These logs record every request, error, warning, and system event. Senior QA engineers read logs to:

- Find errors invisible in the UI
- Investigate production incidents
- Validate correct logging behaviour
- Trace a specific user's journey through the system
- Find performance bottlenecks
- Write Root Cause Analysis (RCA) reports
- Support SRE teams during incidents

**This is a direct requirement in senior QA and QA manager roles.** Production incident support requires reading logs in real time under pressure.

---

## 2. Log Levels

| Level | When used | QA action |
|-------|----------|-----------|
| DEBUG | Detailed developer info | Ignore in production |
| INFO | Normal operations | Read for context |
| WARN | Unexpected but not breaking | Investigate |
| ERROR | Something failed | Fix soon |
| FATAL/CRITICAL | System is broken | Fix immediately — all hands |

---

## 3. Common Log Formats

### Plain Text (Apache/Nginx)
```
192.168.1.1 - - [10/Jun/2026:14:23:45 +0530] 
"POST /api/v1/payment HTTP/1.1" 200 1234
```
Fields: IP, timestamp, method+endpoint, status code, response size

### JSON Structured Logs (modern APIs)
```json
{
  "timestamp": "2026-06-10T14:23:45.123Z",
  "level": "ERROR",
  "service": "payment-service",
  "message": "Payment processing failed",
  "user_id": "USR001",
  "transaction_id": "TXN123456",
  "error": "Insufficient balance",
  "duration_ms": 234,
  "trace_id": "abc123def456"
}
```

### Stack Trace (Java/Node.js)
```
ERROR 2026-06-10 14:23:45 PaymentService — Transaction failed
java.lang.NullPointerException: Cannot invoke method on null
    at PaymentService.processPayment(PaymentService.java:145)
    at PaymentController.handleRequest(PaymentController.java:89)
Caused by: java.lang.IllegalStateException: User not found
    at UserRepository.findById(UserRepository.java:67)
```
Read from top down. Top line = where crash happened. "Caused by" = root cause.

---

## 4. Essential grep Commands for Log Analysis

```bash
# View entire log file
cat app.log

# Count total lines
wc -l app.log

# Find all errors
grep "ERROR" app.log

# Find all fatal errors
grep "FATAL" app.log

# Find errors and fatals together
grep "ERROR\|FATAL" app.log

# Case-insensitive search
grep -i "payment failed" app.log

# Count how many matches
grep -c "ERROR" app.log

# Count errors by service
grep "ERROR" app.log | grep -o '\[.*\]' | sort | uniq -c | sort -rn

# Find specific transaction
grep "TXN003" app.log

# Show 3 lines before and after a match (context)
grep -B3 -A3 "FATAL" app.log

# Find duplicate entries
grep -o "txn_id=[A-Z0-9]*" app.log | sort | uniq -d

# Find slow operations
grep "duration_ms" app.log | grep -o "duration_ms=[0-9]*" | sort -t= -k2 -rn | head -5

# Save filtered output to file
grep "ERROR" app.log > errors_only.log
```

---

## 5. Real Incidents Found in app.log

### Incident 1 — Double Charge Bug 🔴 Critical

**Finding:**
```
08:10:05 Payment initiated TXN003 amount=250 user_id=USR003  ← fires TWICE
08:10:05 Payment initiated TXN003 amount=250 user_id=USR003  ← duplicate
08:10:07 Payment successful TXN003                           ← fires TWICE
08:10:07 Payment successful TXN003                           ← duplicate
08:45:00 ERROR Duplicate transaction detected TXN003
```

**Analysis:**
Transaction TXN003 was initiated and completed twice at the exact same millisecond timestamp. User USR003 was charged ₹250 twice. The system detected the duplicate 35 minutes later — far too late for prevention.

**Root Cause:** No idempotency key implemented on the payment endpoint. The client sent a duplicate request (likely double-tap or network retry) and the server processed both.

**Business Impact:** Customer USR003 charged twice. Requires manual refund. Customer trust damaged. In a high-volume system this could affect thousands of users daily.

**Recommendation:**
- Implement idempotency keys on all payment endpoints
- Add real-time duplicate detection alert — not 35 minutes later
- Add client-side button disable after first tap

---

### Incident 2 — Database Outage 🔴 Fatal

**Timeline from logs:**
```
08:15:00 ERROR [database] Connection pool exhausted — 10/10 connections, 5 waiting
08:15:01 ERROR [payment-service] TXN004 failed — Database unavailable
08:15:02 ERROR [payment-service] TXN004 failed — Database unavailable (retry)
08:15:03 FATAL [database] Database connection lost host=db.internal port=5432
08:20:00 INFO  [database] Database connection restored — down for 5m00s
08:20:01 INFO  [payment-service] TXN004 retry successful
```

**Analysis:**
Database connection pool reached maximum capacity (10/10) at 08:15:00. Within 3 seconds the database connection was completely lost. System was down for exactly 5 minutes. All payment processing failed during this window. TXN004 failed twice before the retry succeeded after restoration.

**Root Cause:** Connection pool exhausted before the database went down — suggesting high traffic or connection leak. The database going down at exactly the same time suggests the pool exhaustion may have caused the database to crash.

**Business Impact:** 5 minutes of payment downtime. All users attempting payments during 08:15–08:20 received failures. Revenue loss and customer experience impact.

**Recommendation:**
- Set alert at 80% connection pool usage — not 100%
- Implement connection pool auto-scaling
- Add circuit breaker pattern — fail fast instead of queuing

---

### Incident 3 — Brute Force Attack 🔴 Security

**Finding:**
```
08:01:15 WARN  Failed login attempt ip=192.168.1.99 attempts=1
08:01:16 WARN  Failed login attempt ip=192.168.1.99 attempts=2
08:01:17 WARN  Failed login attempt ip=192.168.1.99 attempts=3
08:01:18 WARN  Failed login attempt ip=192.168.1.99 attempts=4
08:01:19 ERROR Account locked ip=192.168.1.99 attempts=5
```

**Analysis:**
Automated brute force attack from IP 192.168.1.99. 5 attempts in 4 seconds — humanly impossible. Account was locked after 5 attempts. However 4 WARN lines appeared — alert should have fired at attempt 3 for a fintech application.

**Root Cause:** No IP-based rate limiting, only account-based lockout. Attacker can create multiple accounts and attack all of them.

**Recommendation:**
- Add IP-based rate limiting — block IP after 3 failed attempts across ANY account
- Add CAPTCHA after 2 failed attempts
- Alert security team immediately on brute force pattern detection
- Consider geographic anomaly detection

---

### Incident 4 — Slow Database Query 🟡 Performance

**Finding:**
```
08:25:00 WARN [payment-service] Slow query detected 
duration_ms=3400 
query="SELECT * FROM transactions WHERE user_id=?"
```

**Analysis:**
3.4 second query on the transactions table filtering by user_id. At this speed, loading a user's transaction history takes 3.4 seconds — well above the 2-second threshold for acceptable API response time.

**Root Cause:** Missing database index on user_id column in transactions table. Without an index, the database performs a full table scan — reading every row to find matching user_id values. As the table grows this gets exponentially slower.

**Recommendation:**
- Add index: `CREATE INDEX idx_transactions_user_id ON transactions(user_id)`
- Add composite index if queries also filter by date: `CREATE INDEX idx_transactions_user_date ON transactions(user_id, created_at)`
- Set slow query threshold alert at 1000ms

---

### Incident 5 — Rate Limit Breach 🟡 Security/Performance

**Finding:**
```
08:40:00 WARN  [api-gateway] Rate limit approaching ip=192.168.1.50 requests=95 limit=100
08:40:30 ERROR [api-gateway] Rate limit exceeded ip=192.168.1.50 requests=101 limit=100
```

**Analysis:**
IP 192.168.1.50 hit 95 requests within a 60-second window. A warning was correctly logged. 30 seconds later it exceeded 100 requests and was blocked. The warning-to-block gap of 30 seconds is appropriate.

**Assessment:** Rate limiting is working correctly. The warning-to-block pattern is good practice — gives legitimate high-traffic clients a brief warning before hard blocking.

---

## 6. Log Analysis — Experienced QA vs Fresher

### Fresher approach
"I found a duplicate transaction in the logs — TXN003 appeared twice."

### Senior QA approach
"I identified a double-charge incident affecting user USR003. Transaction TXN003 was initiated and completed twice at identical timestamps — 08:10:05. This indicates either a client-side double submit or a network retry without idempotency checking. The duplicate was detected 35 minutes later — unacceptable for a payment system. I recommend implementing idempotency keys on all payment endpoints, adding a real-time duplicate detection alert with a maximum 30-second window, and disabling the submit button on the client side after first tap."

**The difference:**
- Root cause identified
- Business impact stated
- Concrete recommendations given
- Specific metrics mentioned (35 minutes, 30-second window)

---

## 7. Error Summary — app.log

| Service | Error Count | Root Cause |
|---------|-------------|-----------|
| payment-service | 4 | Database outage (2) + insufficient balance (1) + duplicate (1) |
| auth-service | 2 | Brute force attack — account locked |
| database | 1 | Connection pool exhausted → connection lost |
| api-gateway | 1 | Rate limit exceeded |
| **Total** | **8** | |

**Fatal events:** 1 (database connection lost)
**Security events:** 2 (brute force + rate limit)
**Performance events:** 1 (slow query 3400ms)
**Data integrity events:** 1 (duplicate transaction)

---

## Interview Talking Points

- Can read and analyse backend logs in multiple formats — plain text, JSON, stack traces
- Know all log levels and when each requires action
- Found 5 distinct incidents in a log file — duplicate charge, database outage, brute force, slow query, rate limit breach
- Can write RCA for each incident with root cause, business impact, and recommendations
- Know the difference between showing a finding and explaining the business impact
- Know essential grep commands for log filtering and pattern analysis
- Understand connection pool exhaustion and circuit breaker patterns
- Know idempotency keys and why they prevent duplicate charges in payment systems


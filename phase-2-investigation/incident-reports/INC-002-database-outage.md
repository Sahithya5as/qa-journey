# INC-002 — Database Outage — 5 Minute Payment Service Downtime
**Severity:** Critical
**Status:** Resolved
**Service:** Database + Payment Service
**Start:** 08:15:03
**End:** 08:20:00
**Duration:** 4 minutes 57 seconds

---

## Incident Summary
Database connection pool exhausted at 08:15:00. Database connection completely lost 3 seconds later. Payment service unavailable for 5 minutes. All payment attempts during this window failed.

---

## Timeline
```
08:15:00 — ERROR: Connection pool exhausted (10/10 active, 5 waiting)
08:15:01 — ERROR: TXN004 failed — Database unavailable
08:15:02 — ERROR: TXN004 failed — Database unavailable (retry)
08:15:03 — FATAL: Database connection lost (db.internal:5432)
08:20:00 — INFO: Database restored — down 5m00s
08:20:01 — INFO: TXN004 retry successful
```

---

## Root Cause
No monitoring alert at connection pool threshold. Pool reached 100% capacity before any warning was raised. No circuit breaker to stop new requests when pool was nearly full. Pool exhaustion cascaded into complete database failure.

---

## Business Impact
- 5 minutes complete payment downtime
- All users attempting payments during 08:15–08:20 received errors
- TXN004 failed twice before successful retry

---

## Recommendations
1. Alert at 80% connection pool usage — not 100%
2. Implement circuit breaker pattern
3. Connection pool auto-scaling
4. Runbook for database exhaustion incidents


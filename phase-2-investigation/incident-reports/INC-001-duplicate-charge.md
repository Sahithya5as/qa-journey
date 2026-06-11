# INC-001 — Duplicate Transaction — User Charged Twice
**Severity:** Critical
**Status:** Open
**Service:** Payment Service
**Detected:** Log analysis — 08:45:00
**Occurred:** 08:10:05
**Detection delay:** 34 minutes 55 seconds

---

## Incident Summary
User USR003 was charged ₹250 twice for a single payment intent. The payment service processed two identical requests for transaction TXN003 at the exact same timestamp, resulting in a double debit.

---

## Timeline
```
08:10:05 — Payment initiated TXN003 ₹250 USR003
08:10:05 — Payment initiated TXN003 ₹250 USR003 (DUPLICATE)
08:10:07 — Payment successful TXN003
08:10:07 — Payment successful TXN003 (DUPLICATE)
08:45:00 — ERROR: Duplicate transaction detected (34 mins 55 secs later)
```

---

## Root Cause
No idempotency key implemented on the payment initiation endpoint. Client sent a duplicate request — likely due to network retry or double-tap — and the server processed both independently.

---

## Business Impact
- User USR003 debited ₹250 twice
- Manual refund investigation required
- Customer trust impact
- At scale (100,000 daily transactions, 0.1% duplicate rate) = 100 users overcharged daily

---

## Recommendations
1. Implement idempotency keys on all payment endpoints
2. Real-time duplicate detection alert — maximum 60 second window
3. Client-side button disable after first submission
4. Automated refund trigger when duplicate detected

---

## Evidence
```
grep "TXN003" app.log output:
2026-06-10 08:10:05 INFO  [payment-service] Payment initiated txn_id=TXN003 amount=250 user_id=USR003
2026-06-10 08:10:05 INFO  [payment-service] Payment initiated txn_id=TXN003 amount=250 user_id=USR003
2026-06-10 08:10:07 INFO  [payment-service] Payment successful txn_id=TXN003 duration_ms=456
2026-06-10 08:10:07 INFO  [payment-service] Payment successful txn_id=TXN003 duration_ms=456
2026-06-10 08:45:00 ERROR [payment-service] Duplicate transaction detected txn_id=TXN003
```


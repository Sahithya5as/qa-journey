# Day 14 — Test Cases: Backend Log Analysis
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS Terminal
**Tool:** grep, wc, sort, uniq
**Log file:** app.log — payment service simulation

---

## Execution Summary

| Total | Passed | Failed | Findings |
|-------|--------|--------|----------|
| 20 | 14 | 4 | 5 incidents |

---

## Section 1 — Log File Validation

| TC ID | Scenario | Command | Expected | Actual | Status |
|-------|----------|---------|----------|--------|--------|
| TC_L_001 | Log file created and readable | cat app.log | All log lines visible | 31 lines — all services visible | ✅ Pass |
| TC_L_002 | Total line count matches expected | wc -l app.log | 31 lines | 31 lines confirmed | ✅ Pass |
| TC_L_003 | All services present in logs | grep service names | auth, payment, database, api-gateway | All 4 services present | ✅ Pass |

---

## Section 2 — Error and Fatal Detection

| TC ID | Scenario | Command | Expected | Actual | Status |
|-------|----------|---------|----------|--------|--------|
| TC_L_004 | All errors captured | grep ERROR | 8 error lines | 8 errors found | ✅ Pass |
| TC_L_005 | Fatal errors isolated | grep FATAL | 1 fatal | Database connection lost | ✅ Pass |
| TC_L_006 | Error count by service | grep + uniq -c | Service breakdown | payment-service:4, auth:2, db:1, gateway:1 | ✅ Pass |
| TC_L_007 | No unlogged errors | Cross-check INFO vs ERROR | All failures logged | All payment failures have corresponding ERROR | ✅ Pass |

---

## Section 3 — Payment Integrity Checks

| TC ID | Scenario | Command | Expected | Actual | Status |
|-------|----------|---------|----------|--------|--------|
| TC_L_008 | No duplicate transactions | grep txn_id + uniq -d | No duplicates | TXN003 appears as duplicate | ⚠️ Fail — Bug found |
| TC_L_009 | Duplicate detected within 60 seconds | Check timestamps | Alert within 60s | Detected after 35 minutes | ⚠️ Fail — Too slow |
| TC_L_010 | Failed payments logged with reason | grep Payment failed | Error reason present | Insufficient balance + DB unavailable logged | ✅ Pass |
| TC_L_011 | Payment retry after failure | Check TXN004 lifecycle | Retry attempted | TXN004 retried and succeeded after DB restored | ✅ Pass |
| TC_L_012 | Payment duration logged | grep duration_ms | All payments have timing | All successful payments have duration_ms | ✅ Pass |

---

## Section 4 — Security Event Checks

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_L_013 | Brute force attempts logged | WARN on each attempt | 4 WARN lines logged | ✅ Pass |
| TC_L_014 | Account locked after 5 attempts | ERROR on lock | Account locked logged | ✅ Pass |
| TC_L_015 | Brute force IP logged | IP visible in log | 192.168.1.99 logged | ✅ Pass |
| TC_L_016 | Lock threshold appropriate for fintech | Should lock at 3 for payment apps | Locks at 5 — too permissive | ⚠️ Fail |
| TC_L_017 | Rate limit warning before block | WARN before ERROR | Warning at 95/100 — 30s before block | ✅ Pass |

---

## Section 5 — Performance Checks

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_L_018 | No queries over 1000ms | All queries under 1s | Slow query 3400ms detected | ⚠️ Fail |
| TC_L_019 | Database outage logged | FATAL + recovery logged | Full outage timeline logged | ✅ Pass |
| TC_L_020 | Database recovery logged with downtime duration | duration_down in log | duration_down=5m00s logged | ✅ Pass |

---

## Incident Summary Found via Log Analysis

| Incident | Severity | Detection Time | Business Impact |
|----------|----------|---------------|-----------------|
| Duplicate charge — TXN003 | Critical | 35 minutes | User charged twice |
| Database outage — 5 mins | Critical | Immediate | All payments failed |
| Brute force attack | High | Immediate | Account locked |
| Slow query — 3400ms | Medium | Immediate | Slow transaction history |
| Rate limit breach | Medium | Immediate | IP blocked |


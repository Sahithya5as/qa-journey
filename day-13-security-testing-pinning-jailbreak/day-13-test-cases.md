# Day 13 — Test Cases: Security Testing — Certificate Pinning + Jailbreak Detection
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS + LAVA LZG409 (Android 14) + iPhone 14 (iOS 26.2.1)
**Tools:** Charles Proxy + ADB
**Application:** PhonePe (com.phonepe.app)

---

## Execution Summary

| Total | Passed | Failed | Observations |
|-------|--------|--------|--------------|
| 15 | 12 | 0 | 3 |

---

## Section 1 — Certificate Pinning Verification

| TC ID | Scenario | Steps | Expected | Actual | Status |
|-------|----------|-------|----------|--------|--------|
| TC_S_001 | Certificate pinning blocks Charles on Android | Set Charles proxy → open PhonePe | SSL handshake failure — traffic not decrypted | SSL handshake failed — certificate_unknown | ✅ Pass |
| TC_S_002 | All PhonePe domains are pinned | Check multiple domains in Charles | All domains show as unknown | apicp1.phonepe.com and imgstatic.phonepe.com both unknown | ✅ Pass |
| TC_S_003 | SSL error message is specific | Check Charles failure message | certificate_unknown or similar specific error | certificate_unknown confirmed | ✅ Pass |
| TC_S_004 | App continues working despite proxy | Use PhonePe normally | App functions normally — connects directly to server | App worked normally | ✅ Pass |

---

## Section 2 — Debug Mode Verification

| TC ID | Scenario | Command | Expected | Actual | Status |
|-------|----------|---------|----------|--------|--------|
| TC_S_005 | Production build not debuggable | adb shell run-as com.phonepe.app | package not debuggable error | run-as: package not debuggable: com.phonepe.app | ✅ Pass |
| TC_S_006 | Private data directory not accessible | adb shell run-as → ls | Permission denied or not debuggable | Not debuggable — access blocked | ✅ Pass |
| TC_S_007 | No debug flags in package info | adb shell dumpsys package grep debuggable | No debuggable=true flag | No debug flags found | ✅ Pass |

---

## Section 3 — Jailbreak Detection (iOS — from Day 12 logs)

| TC ID | Scenario | Evidence Source | Expected | Actual | Status |
|-------|----------|----------------|----------|--------|--------|
| TC_S_008 | Jailbreak detection implemented | Xcode Console logs | Sandbox deny logs present | deny file-test-existence /bin/bash confirmed | ✅ Pass |
| TC_S_009 | Multiple jailbreak indicators checked | Xcode Console logs | Multiple paths checked | /bin/bash and /private/etc/apt both checked | ✅ Pass |
| TC_S_010 | Detection runs on every launch | Xcode Console | Fires each app open | Confirmed on multiple launches | ✅ Pass |
| TC_S_011 | No false positive on clean device | Zomato works normally | App functions on non-jailbroken device | App worked normally — no false positive | ✅ Pass |

---

## Section 4 — Security Observations

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_S_012 | PhonePe does not expose auth token in URL | Token in header only | Not verifiable — pinning blocks Charles | 👁️ Observation — requires debug build |
| TC_S_013 | PhonePe traffic over HTTPS only | No HTTP endpoints | All visible traffic is port 443 | ✅ All HTTPS |
| TC_S_014 | TLS version is current | TLS 1.2 minimum | TLS 1.3 confirmed | ✅ Pass |
| TC_S_015 | Android ACL blocks unauthorised devices | Unknown IPs blocked | 192.168.1.5 denied until ACL updated | 👁️ Observation — expected Charles behaviour |


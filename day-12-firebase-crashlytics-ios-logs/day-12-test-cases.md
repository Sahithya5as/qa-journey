# Day 12 — Test Cases: iOS Device Log Analysis
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS + iPhone 14 (iOS 26.2.1)
**Tool:** Xcode Console
**Application:** Zomato iOS

---

## Execution Summary

| Total | Passed | Failed | Observations |
|-------|--------|--------|--------------|
| 18 | 11 | 3 | 4 |

---

## Section 1 — Xcode Console Setup

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_X_001 | iPhone detected in Xcode Devices | Device listed with model and iOS version | iPhone 14, iOS 26.2.1 detected | ✅ Pass |
| TC_X_002 | Console streams live logs | Real-time logs visible | 1,41,152 messages streaming | ✅ Pass |
| TC_X_003 | Developer Mode enables deeper logging | Private values visible after enabling | Developer Mode enabled | ✅ Pass |
| TC_X_004 | WiFi connection works without USB | Logs stream over WiFi | Connected via network — no USB needed | ✅ Pass |
| TC_X_005 | Errors and Faults filter works | Only error/fault messages shown | Filter working correctly | ✅ Pass |

---

## Section 2 — Jailbreak Detection Testing

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_X_006 | Jailbreak detection implemented | App checks for jailbreak indicators | Sandbox deny logs confirm checks | ✅ Pass |
| TC_X_007 | /bin/bash check present | App checks for Unix shell | deny(2) file-test-existence /bin/bash logged | ✅ Pass |
| TC_X_008 | /private/etc/apt check present | App checks for Cydia | deny(2) file-test-existence /private/etc/apt logged | ✅ Pass |
| TC_X_009 | Detection runs on every launch | Not just first launch | Confirmed — fires on each app open | ✅ Pass |
| TC_X_010 | No false positive on normal device | App works normally on non-jailbroken device | App functioned normally | ✅ Pass |

---

## Section 3 — App Tracking Transparency

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_X_011 | ATT status checked on launch | App checks tracking permission | trackingAuthorizationStatus invoked | ✅ Pass |
| TC_X_012 | ATT denial respected | App does not track when denied | Status 2 (Denied) — correct | ✅ Pass |
| TC_X_013 | ATT rate limiting applied | Cannot call API too frequently | Rate limiting confirmed in logs | ✅ Pass |

---

## Section 4 — Database and Cache Issues

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_X_014 | SQLite PRAGMA query executes correctly | No syntax errors in database queries | Syntax error: PRAGMA user_version = ? | ⚠️ Fail |
| TC_X_015 | Cached data deserialises without errors | NSKeyedUnarchiver works cleanly | 15+ NSKeyedUnarchiver errors on launch | ⚠️ Fail |
| TC_X_016 | No repeated error patterns | Each error unique | Same NSKeyedUnarchiver error repeated 15+ times | ⚠️ Fail |

---

## Section 5 — Sensor and Background Activity

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_X_017 | Location tracking proportional to need | Only GPS — no excessive sensors | Step count, altimeter, motion all active | 👁️ Observation |
| TC_X_018 | NFC daemon activity normal | Occasional NFC updates | 12 camera state updates in 2 seconds | 👁️ Observation |


# Day 12 — Bug Reports: iOS Log Analysis
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** iPhone 14 — iOS 26.2.1
**Tool:** Xcode Console
**Application:** Zomato iOS

---

## Summary

| Bug ID | Title | Severity | Priority | Found By |
|--------|-------|----------|----------|----------|
| BUG-X001 | SQLite syntax error in PRAGMA query on app launch | Medium | Medium | Xcode Console |
| BUG-X002 | NSKeyedUnarchiver errors repeated 15+ times on launch — cache data migration issue | Medium | Medium | Xcode Console |
| BUG-X003 | Excessive sensor usage — step count altimeter and motion active continuously | Low | Low | Xcode Console |
| BUG-X004 | NFC daemon camera state updated 12 times in 2 seconds — unnecessary polling | Low | Low | Xcode Console |

---

## BUG-X001

**Title:** Zomato iOS — SQLite syntax error in PRAGMA user_version query on app launch — database migration issue — iPhone 14 iOS 26.2.1

**Severity:** Medium
**Priority:** Medium
**Type:** Functional — Database
**Found by:** Xcode Console — filtered by Zomato — Errors and Faults view

**Log Evidence:**
```
18:26:34.833939+0530  Zomato  near "?": syntax error in "PRAGMA user_version = ?"
18:26:34.833945+0530  Zomato  near "?": syntax error in "PRAGMA user_version = ?"
```

**Expected Result:**
SQLite PRAGMA statements should execute without syntax errors. `PRAGMA user_version` is used to track database schema version for migrations. It should execute cleanly on every launch.

**Actual Result:**
The query `PRAGMA user_version = ?` fails with a syntax error because the `?` placeholder is not being bound to a value before execution. The query fires twice on every launch.

**Impact:**
Medium — if database version tracking fails, schema migrations may not run correctly. This could cause data corruption or missing features after an app update. Appearing twice on every launch suggests this is a persistent issue affecting all users on this iOS version.

**Technical Root Cause:**
SQLite does not support parameter binding (`?` placeholders) in PRAGMA statements. The developer is incorrectly using a prepared statement with a placeholder for a PRAGMA query. The fix is to format the version number directly into the SQL string.

---

## BUG-X002

**Title:** Zomato iOS — NSKeyedUnarchiver validation errors repeated 15+ times on app launch — cached data deserialisation failing — iPhone 14 iOS 26.2.1

**Severity:** Medium
**Priority:** Medium
**Type:** Performance — Cache Management
**Found by:** Xcode Console — Errors and Faults filter

**Log Evidence:**
```
*** -[NSKeyedUnarchiver validateAllowedClass:forKey:] 
allowed unarchiving safe plist type 'NSString' (0x20464d86...)
```
Repeated 15+ times within 10ms of app launch.

**Expected Result:**
Cached data should deserialise cleanly with no errors. Each NSKeyedUnarchiver call should succeed silently.

**Actual Result:**
15+ consecutive NSKeyedUnarchiver validation errors fire within 10 milliseconds of app launch. The app is attempting to unarchive cached data from a previous format that no longer matches the current expected types.

**Impact:**
Medium — 15+ errors on every launch indicate:
- Cached data from previous app version is incompatible with current version
- App is spending processing time on failed deserialisation
- Data that should be loaded from cache may be falling back to network requests — causing slower initial load
- Potential for data loss if cached user preferences or session data cannot be read

**Root Cause:**
App update changed data models without migrating or invalidating existing cached data. Old cache remains on device in previous format. NSKeyedUnarchiver cannot validate the old types against new class definitions.

---

## BUG-X003

**Title:** Zomato iOS — Excessive sensor usage on background — step counter barometric altimeter and motion state all active continuously — battery drain concern — iPhone 14 iOS 26.2.1

**Severity:** Low
**Priority:** Low
**Type:** Performance — Battery
**Found by:** Xcode Console — locationd process logs

**Log Evidence:**
```
locationd: CLStepCountNotifier::onMotionStateMediatorNotification
locationd: CLBarometricAltimeter::onOdometerUpdate
locationd: CLMotionStateObserver::onMotionStateMediatorNotification
locationd: CLCatherineNotifier::onMotionStateMediatorNotification
```
All firing continuously during normal app use.

**Expected Result:**
A food delivery app needs GPS location for delivery address and restaurant discovery. Step counting, barometric altitude, and odometer tracking are not required for food delivery functionality.

**Actual Result:**
Zomato is accessing step counter, barometric altimeter, and motion state sensors continuously. These are fitness/health sensors not related to food delivery.

**Impact:**
Low-Medium — continuous access to multiple sensors:
- Increases battery drain
- Raises user privacy concerns — why does a food app need step count and altitude?
- May affect app performance on older devices
- Could raise regulatory concerns under India's DPDP Act regarding proportionate data collection

---

## BUG-X004

**Title:** Zomato iOS — NFC daemon camera state updated 12 times in 2 seconds — excessive polling frequency — iPhone 14 iOS 26.2.1

**Severity:** Low
**Priority:** Low
**Type:** Performance
**Found by:** Xcode Console — nfcd process logs

**Log Evidence:**
```
19:06:03.621125+0530  nfcd  Camera state updated
19:06:03.621373+0530  nfcd  Camera state updated
19:06:03.621447+0530  nfcd  Camera state updated
[... 9 more identical entries within 1 second]
```
12 identical entries within approximately 1 second.

**Expected Result:**
Camera state should update only when the camera state actually changes — not continuously poll at high frequency.

**Actual Result:**
NFC daemon triggered 12 camera state updates within 2 seconds. This polling frequency is excessive and suggests an event listener is not being properly debounced or the NFC session is not being managed correctly.

**Impact:**
Low — excessive NFC daemon activity contributes to battery drain and unnecessary CPU usage. On devices with older batteries this could cause noticeable battery degradation during extended Zomato usage.


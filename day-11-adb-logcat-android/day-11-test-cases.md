# Day 11 — Test Cases: ADB Logcat Investigation
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS + LAVA LZG409 (Android 14, SDK 34)
**Tool:** ADB (Android Debug Bridge)
**Application:** PhonePe (com.phonepe.app)

---

## Execution Summary

| Total | Passed | Failed | Observations |
|-------|--------|--------|--------------|
| 18 | 12 | 2 | 4 |

---

## Section 1 — ADB Setup and Device Connection

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_A_001 | ADB installed successfully | adb version returns version number | Android Debug Bridge version 1.0.41 | ✅ Pass |
| TC_A_002 | Device recognised via USB | adb devices shows device as authorised | LVO3FK24LB017102 device — connected | ✅ Pass |
| TC_A_003 | Developer options enabled | USB debugging active | USB debugging enabled on LAVA LZG409 | ✅ Pass |
| TC_A_004 | Logcat captures live logs | adb logcat streams real-time output | Live logs streaming correctly | ✅ Pass |

---

## Section 2 — Log Level Filtering

| TC ID | Scenario | Command | Expected | Actual | Status |
|-------|----------|---------|----------|--------|--------|
| TC_A_005 | Error filter shows only errors | adb logcat *:E | Only E level lines | Error lines only shown | ✅ Pass |
| TC_A_006 | App-specific filter works | adb logcat grep com.phonepe.app | Only PhonePe log lines | PhonePe lines captured | ✅ Pass |
| TC_A_007 | Log clear works | adb logcat -c | Old logs cleared | Logs cleared successfully | ✅ Pass |
| TC_A_008 | Logs saved to file | adb logcat > logs.txt | File created with log content | File created and pulled | ✅ Pass |

---

## Section 3 — Memory Analysis

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_A_009 | PhonePe memory usage acceptable | Private Dirty under 100MB | Private Dirty: high — 254MB RSS total | ⚠️ Observation |
| TC_A_010 | No SwapPss pressure | SwapPss Dirty under 10MB | SwapPss Dirty: 24-27MB — memory pressure | ⚠️ Fail |
| TC_A_011 | APK size in memory acceptable | .apk mmap under 50MB | .apk mmap: 108MB — very large | ⚠️ Fail |
| TC_A_012 | GC pause under 100ms | GC pause under 100ms | GC pause: 482ms — nearly half second | ⚠️ Observation |
| TC_A_013 | Heap usage healthy | Heap free above 30% | 47% free after GC — borderline | ⚠️ Observation |

---

## Section 4 — Device and Process Info

| TC ID | Scenario | Command | Expected | Actual | Status |
|-------|----------|---------|----------|--------|--------|
| TC_A_014 | Device model retrieved | getprop ro.product.model | Device model returned | LAVA LZG409 | ✅ Pass |
| TC_A_015 | Android version retrieved | getprop ro.build.version.release | Version number returned | 14 | ✅ Pass |
| TC_A_016 | SDK version retrieved | getprop ro.build.version.sdk | SDK number returned | 34 | ✅ Pass |
| TC_A_017 | Battery info retrieved | dumpsys battery | Battery state returned | Charging via USB, health: Good | ✅ Pass |
| TC_A_018 | Screenshot captured remotely | adb shell screencap + adb pull | Screenshot saved to Mac Desktop | 227KB PNG pulled at 18.1 MB/s | ✅ Pass |


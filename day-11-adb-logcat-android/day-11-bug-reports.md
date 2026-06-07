# Day 11 — Bug Reports: ADB Investigation
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** LAVA LZG409 — Android 14 — SDK 34
**Tool:** ADB logcat + dumpsys meminfo
**Application:** PhonePe (com.phonepe.app)

---

## Summary

| Bug ID | Title | Severity | Priority | Found By |
|--------|-------|----------|----------|----------|
| BUG-A001 | PhonePe GC pause 482ms — app freezes during garbage collection on budget device | Medium | High | ADB logcat |
| BUG-A002 | PhonePe APK memory mapping 108MB — excessive memory on low-end devices | Medium | Medium | ADB dumpsys meminfo |
| BUG-A003 | SwapPss pressure 24-27MB — device memory insufficient for PhonePe — LAVA LZG409 | Medium | Medium | ADB dumpsys meminfo |
| BUG-A004 | App-specific configuration missing for PhonePe on LAVA device — W level warning | Low | Low | ADB logcat |

---

## BUG-A001

**Title:** PhonePe Android — Garbage Collection pause of 482ms causes app freeze on LAVA LZG409 (Android 14) — performance degradation during payment flows

**Severity:** Medium
**Priority:** High
**Type:** Performance — Memory Management
**Device:** LAVA LZG409, Android 14, SDK 34
**Found by:** ADB logcat — grep com.phonepe.app

**Log Evidence:**
```
I com.phonepe.app: Background concurrent mark compact GC freed 
12MB AllocSpace bytes, 224(14MB) LOS objects, 47% free, 
26MB/50MB, paused 7.843ms,12.240ms total 482.588ms
```

**Expected Result:**
GC pauses should be under 100ms. Modern Android GC should run concurrently with minimal pause time. Users should not experience visible freezes during normal app usage.

**Actual Result:**
Total GC pause of 482ms observed — the app froze for nearly half a second during garbage collection. The pause consisted of two stop-the-world pauses of 7.843ms and 12.240ms plus 482ms total GC duration.

**Impact:**
Medium-High — a 482ms freeze in a payment application is particularly concerning. If this occurs during:
- OTP entry — user may mis-tap
- Payment confirmation — user may think app has crashed and tap again causing duplicate payment
- Transaction processing — user may see a frozen screen and force close the app

**Root Cause:**
Memory allocation pressure on a budget device with limited RAM. The app is allocating and releasing large objects (14MB LOS objects) causing frequent GC cycles.

**Recommendation:**
- Profile memory allocations in PhonePe on low-end devices
- Reduce large object allocations in critical payment flows
- Consider implementing object pooling for frequently allocated objects
- Test specifically on devices with 2GB RAM or less

---

## BUG-A002

**Title:** PhonePe Android — APK memory mapping of 108MB exceeds acceptable threshold for budget devices — LAVA LZG409 Android 14

**Severity:** Medium
**Priority:** Medium
**Type:** Performance — Memory Usage
**Device:** LAVA LZG409, Android 14, SDK 34
**Found by:** ADB dumpsys meminfo com.phonepe.app

**Memory Evidence:**
```
           Pss    Private  Private  SwapPss
         Total    Dirty    Clean    Dirty
.apk mmap  108805   284    101,404    0
```

**Expected Result:**
APK memory mapping should be under 50MB for a mobile payment app. The app should use code splitting and dynamic feature modules to load only necessary code.

**Actual Result:**
108MB of the APK is mapped into memory — 101MB as private clean pages. This means 108MB of device RAM is consumed just for the app code, before any data or UI is loaded.

**Impact:**
Medium — on a budget device like LAVA LZG409 with limited RAM, 108MB for APK mapping alone pushes the device into memory pressure (SwapPss). Combined with heap usage the total RSS is 254MB — consuming a significant portion of available device RAM.

**Recommendation:**
- Implement dynamic feature modules — load payment features only when needed
- Enable R8/ProGuard code shrinking aggressively
- Use Android App Bundle to reduce installed APK size
- Profile which classes are loaded on startup vs on demand

---

## BUG-A003

**Title:** PhonePe — SwapPss Dirty 24-27MB indicates device memory pressure — app pages swapped to storage causing performance degradation — LAVA LZG409

**Severity:** Medium
**Priority:** Medium
**Type:** Performance — Memory Pressure
**Device:** LAVA LZG409, Android 14, SDK 34
**Found by:** ADB dumpsys meminfo

**Memory Evidence:**
```
                  SwapPss Dirty
Native Heap:      24,086 KB = 24MB
Dalvik Heap:      27,614 KB = 27MB
Dalvik Other:      6,357 KB
Stack:             4,568 KB
Total swap:       ~62MB pushed to storage
```

**Expected Result:**
SwapPss Dirty should be near 0MB on a healthy device. Memory should fit in RAM without being swapped to storage.

**Actual Result:**
62MB of PhonePe's memory has been pushed to device storage swap. Reading from storage is 10-100x slower than reading from RAM. Every time the OS needs this memory it must read from storage first.

**Impact:**
Medium — swap usage causes:
- Slower screen transitions
- Delayed response to taps
- Slower payment processing UI
- Increased battery drain (storage I/O is power-intensive)

This is particularly impactful for a fintech app where users expect instant response during transactions.

---

## BUG-A004

**Title:** PhonePe — App-specific configuration missing for LAVA LZG409 — W level warning in logcat — may cause display rendering differences

**Severity:** Low
**Priority:** Low
**Type:** Compatibility
**Device:** LAVA LZG409, Android 14
**Found by:** ADB logcat — W level filter

**Log Evidence:**
```
W PackageConfigPersister: App-specific configuration not found 
for packageName: com.phonepe.app and userId: 0
```

**Expected Result:**
No configuration warnings should appear in logcat for any supported device.

**Actual Result:**
Warning logged indicating PhonePe has no device-specific configuration for this LAVA device. Display behaviour may fall back to defaults which could differ from intended design.

**Impact:**
Low — may cause minor display differences on LAVA devices. Font rendering, colour profiles, or display density may not be optimised for this specific device model.


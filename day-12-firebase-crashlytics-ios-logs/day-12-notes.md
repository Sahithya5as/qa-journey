# Day 12 — Firebase Crashlytics + iOS Device Logs
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS + iPhone 14 (iOS 26.2.1)
**Tool:** Xcode Console + idevicesyslog
**Application:** Zomato iOS

---

## 1. Firebase Crashlytics — What Every Senior QA Must Know

### What is Firebase Crashlytics
Firebase Crashlytics is a real-time crash reporting tool built by Google. When an app crashes in production, Crashlytics automatically captures the full stack trace, device information, OS version, app version, user actions before the crash, and custom developer logs.

**Why QA engineers need Crashlytics:**
Senior QA engineers are not just testers — they are quality monitors. After every release you watch Crashlytics for crash spikes. A new build that increases crash rate by 0.1% could affect thousands of users. You catch it before the PM even knows.

### Key Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| Crash-free users % | % of users who did NOT crash | > 99.5% |
| Crash rate | Crashes per 1000 sessions | < 1 |
| ANR rate | App Not Responding per 1000 sessions | < 0.47 |
| Non-fatal error rate | Logged errors that did not crash app | Monitor trend |

### Reading a Crashlytics Stack Trace

```
Fatal Exception: java.lang.NullPointerException
Attempt to invoke virtual method on a null object reference

com.phonepe.app.payment.PaymentFragment.processPayment(PaymentFragment.kt:247)
com.phonepe.app.payment.PaymentFragment.onConfirmClicked(PaymentFragment.kt:189)
com.phonepe.app.payment.PaymentViewModel.handleConfirm(PaymentViewModel.kt:134)
```

**How to read:**
- Line 1: Exception type — NullPointerException
- Line 2: What happened — tried to use a null object
- Line 4: Where it crashed — PaymentFragment.kt line 247
- Lines 4-6: Call chain — user tapped Confirm → handleConfirm → processPayment → crash

**QA process when receiving a Crashlytics alert:**
1. Identify exception type and location
2. Check affected user count and crash rate
3. Identify which app version introduced the crash
4. Reproduce on a device using the exact steps
5. Raise bug report with stack trace attached
6. Monitor after fix — verify crash rate returns to baseline

### Non-Fatal Errors vs Crashes vs ANR

| Type | Definition | Example | QA action |
|------|-----------|---------|-----------|
| Fatal crash | App terminates | NullPointerException in payment | Immediate P1 bug |
| Non-fatal error | Error logged, app continues | Network timeout | Monitor — may escalate |
| ANR | App frozen 5+ seconds | Main thread blocked | Investigate performance |

### Crashlytics for iOS — Symbolication
iOS crash reports contain memory addresses instead of readable code. Symbolication converts addresses to human-readable class names and line numbers using the dSYM file (debug symbols). Without symbolication crash reports look like:

```
0x00000001045a3f4c MyApp + 1234567
```

With symbolication:
```
PaymentViewController.confirmPayment() PaymentViewController.swift:145
```

QA engineers need to understand symbolication because unsymbolicated crashes in Crashlytics mean the dSYM was not uploaded — a process gap to flag.

---

## 2. Xcode Console — iOS Device Log Analysis

### Setup
- Device: iPhone 14, iOS 26.2.1 (23C71)
- Connected: WiFi (Connect via Network — established after initial USB trust)
- Developer Mode: Enabled via Settings → Privacy & Security → Developer Mode

### How Xcode Detects iPhone Automatically
When iPhone connects via USB, macOS runs a background service called `AMDMobileDeviceService` (Apple Mobile Device Service). This service captures the device UDID, model, iOS version, and name. Xcode reads from this service and displays the device automatically. After the initial USB trust is established, Xcode can connect wirelessly over WiFi — both devices must be on the same network.

### Console Interface

| Section | What it contains |
|---------|-----------------|
| All Messages | Every log line from every process |
| Errors and Faults | Filtered — only errors and crashes |
| Crash Reports | Historical crash reports from all apps |
| Spin Reports | ANR equivalent — main thread blocked |
| Log Reports | System logs |
| Diagnostic Reports | Battery, performance diagnostics |

### Filtering Logs — How to Control the Flood
iOS logs 1,41,000+ messages per session. Filtering is essential.

**Step 1:** Click Pause (⏸) or Stop (✕)
**Step 2:** Press Cmd + K to clear
**Step 3:** Type app name in Search box
**Step 4:** Click "Errors and Faults" button
**Step 5:** Resume — only filtered messages stream

**Log dot colours:**
- ⚫ Grey dot = Info message
- 🟡 Yellow dot = Warning
- 🔴 Red dot = Error or Fault

---

## 3. Real Findings from Zomato iOS — Xcode Console

### Finding 1 — Jailbreak Detection Confirmed ✅ Security Feature
```
kernel: Sandbox: Zomato(677) deny(2) file-test-existence /bin/bash
kernel: Sandbox: Zomato(677) deny(2) file-test-existence /private/etc/apt
```

Zomato checks for jailbreak indicators on every launch:
- `/bin/bash` — Unix shell, present on jailbroken devices
- `/private/etc/apt` — Cydia package manager, only on jailbroken iPhones

The iOS kernel sandbox blocks these file checks and logs them. This proves jailbreak detection is implemented. For a fintech or food delivery app handling payments, jailbreak detection is a security requirement — jailbroken devices can bypass certificate pinning, intercept tokens, and modify app behaviour.

**QA validation required:**
- Verify app shows warning on jailbroken device
- Verify payment features are blocked on jailbroken device
- Verify the check runs on every launch — not just first launch

### Finding 2 — App Tracking Transparency Status
```
[ATTrackingManager] trackingAuthorizationStatus API call invoked
[ATTrackingManager] Call to trackingAuthorizationStatus eligible for rate limiting. Returning 2
```

Status 2 = `ATTrackingManagerAuthorizationStatusDenied` — tracking permission denied by user. App correctly checks ATT status on launch. Rate limiting is applied — app cannot call this API too frequently.

### Finding 3 — SQLite Syntax Error 🔴 Bug
```
near "?": syntax error in "PRAGMA user_version = ?"
```
Appeared twice. Zomato's local SQLite database query has a syntax error. The `?` placeholder is not being bound correctly in the PRAGMA statement. This is a database migration issue.

### Finding 4 — NSKeyedUnarchiver Repeated Errors 🔴 15+ errors
```
*** -[NSKeyedUnarchiver validateAllowedClass:forKey:] 
allowed unarchiving safe plist type 'NSString'
```
15+ repeated errors during app launch. NSKeyedUnarchiver is deserialising cached data from previous app versions. The repeated `validateAllowedClass` calls suggest a data migration issue — old cached data format not matching current expected format.

### Finding 5 — Network Connection Establishment
```
nw_endpoint_resolver_update [C36.1.1 Hostname#3d712dd2:443 in_progress]
[C36.1.1.1 IPv6#d2c1aa7f.443 initial path ((null))] event: path:start @0.053s
[C36.1.1.1 IPv6#d2c1aa7f.443 waiting path (satisfied)]
```
Network path establishment to Zomato API server. Connection satisfied via en0 (WiFi), port 443 (HTTPS). IPv6 used. Connection established within 0.053 seconds.

### Finding 6 — Location and Motion Sensors Active
```
locationd: CLLocationController::onMotionStateMediatorNotification
locationd: CLMotionStateMediator::onStepCountNotification
locationd: CLBarometricAltimeter::onOdometerUpdate
```
Zomato is continuously tracking step count, barometric altitude, and motion state in the background. This is more sensor data than needed for a food delivery app. Raises battery drain concerns.

### Finding 7 — NFC Daemon Activity
```
nfcd: Camera state updated (×12 in rapid succession)
```
NFC daemon updating camera state 12 times in 2 seconds. Unusual frequency — possible unnecessary polling.

---

## 4. Jailbreak Detection — Complete QA Guide

### What is Jailbreak
Jailbreaking removes Apple's security restrictions from an iPhone. On a jailbroken device:
- Apps can access any file on the system
- Unauthorised apps install via Cydia
- SSL certificate pinning can be bypassed
- App binaries can be modified
- Charles Proxy decrypts ALL app traffic

### Why Fintech Apps Must Detect Jailbreak

| Risk | Impact |
|------|--------|
| Certificate pinning bypass | HTTPS traffic interceptable |
| Token theft | Auth tokens readable from memory |
| Transaction modification | Payment amounts changeable |
| App binary modification | Security checks removable |
| Credential theft | Login credentials extractable |

### How to Test Jailbreak Detection
1. Test on a real jailbroken device (lab device)
2. Use iOS Simulator with jailbreak indicator files planted
3. Use Frida dynamic instrumentation framework
4. Check logs for Sandbox deny messages (as found today)

### QA Test Cases for Jailbreak Detection
- App should show warning on jailbroken device
- App should block login or payment on jailbroken device
- Detection should run on every launch — not just first
- Detection should not produce false positives on normal devices

---

## 5. Key Concepts — iOS Logs vs Android Logs

| Aspect | iOS (Xcode Console) | Android (ADB Logcat) |
|--------|--------------------|--------------------|
| Tool | Xcode Console / idevicesyslog | adb logcat |
| Connection | USB or WiFi | USB only (default) |
| Log volume | Extremely high | High |
| Filtering | Search box + Errors and Faults | *:E, grep |
| Crash reports | Xcode left panel | logcat FATAL EXCEPTION |
| Privacy | Values shown as <private> | Values visible |
| Jailbreak/Root | Sandbox deny logs | Similar SELinux logs |

---

## Interview Talking Points

- Can explain Firebase Crashlytics metrics — crash-free users, crash rate, ANR rate
- Know how to read a stack trace and identify crash location
- Understand symbolication and why unsymbolicated crashes are a process gap
- Set up Xcode Console with iPhone over WiFi — no USB cable needed after initial trust
- Found jailbreak detection implementation in Zomato via Sandbox deny logs
- Know why fintech apps must detect jailbreak — certificate pinning, token theft
- Found SQLite syntax error in production app from iOS logs
- Found NSKeyedUnarchiver repeated errors — data migration issue
- Know how to filter 141,000+ log messages to find relevant entries
- Can explain ATTrackingManager and App Tracking Transparency


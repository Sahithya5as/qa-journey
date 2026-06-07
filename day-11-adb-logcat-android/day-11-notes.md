# Day 11 — ADB Logcat — Android Log Analysis
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS + LAVA LZG409 (Android 14, SDK 34)
**Tool:** ADB (Android Debug Bridge)
**Application Tested:** PhonePe (com.phonepe.app)

---

## What is ADB

ADB (Android Debug Bridge) is a command-line tool that creates a communication channel between your Mac and an Android device. It is part of the Android SDK Platform Tools.

With ADB you can:
- Read real-time device logs (logcat)
- Install and uninstall APKs without the Play Store
- Copy files between Mac and device
- Take screenshots and screen recordings
- Run shell commands directly on the device
- Simulate hardware events (button presses, swipes)
- Check memory, battery, and process information

**Why QA engineers need ADB:**
When Charles Proxy cannot decrypt an app's traffic (certificate pinning on Android 7+), ADB logcat is the next investigation tool. The app's own log statements, crash reports, network errors, and debug messages all appear in logcat — even for apps that block Charles.

---

## Setup Completed Today

### Installation
```bash
brew install android-platform-tools
adb version
# Android Debug Bridge version 1.0.41
```

### Device Connected
```bash
adb devices
# LVO3FK24LB017102  device
```

### Developer Options enabled on LAVA LZG409
Settings → About phone → Build number (tapped 7 times) → USB debugging enabled

---

## Device Information Captured

```
Model:           LAVA LZG409
Android version: 14
SDK version:     34
Power:           USB powered (charging via Mac)
Battery status:  2 = Charging
Battery health:  2 = Good
Max charge:      1.15A at 5V
```

---

## ADB Logcat — Complete Guide

### Log Levels

| Level | Code | Meaning | When you see it |
|-------|------|---------|-----------------|
| Verbose | V | Everything | Development only |
| Debug | D | Debug messages | Dev builds |
| Info | I | General information | Normal operation |
| Warning | W | Something might be wrong | Investigate |
| Error | E | Something went wrong | Always investigate |
| Fatal | F | App crashed | Immediate action |

### Essential Commands

```bash
# Check device connected
adb devices

# See ALL logs live
adb logcat

# Errors only — most useful for QA
adb logcat *:E

# Warnings and above
adb logcat *:W

# Clear old logs before testing
adb logcat -c

# Save logs to file
adb logcat > ~/Desktop/logs.txt

# Filter by app package name
adb logcat | grep "com.phonepe.app"

# Find crashes specifically
adb logcat | grep -E "FATAL|Exception|ANR|crash"

# Take screenshot
adb shell screencap /sdcard/screenshot.png
adb pull /sdcard/screenshot.png ~/Desktop/

# Device info
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk

# Battery info
adb shell dumpsys battery

# Memory info for specific app
adb shell dumpsys meminfo com.phonepe.app

# List all installed apps
adb shell pm list packages -3

# Running processes
adb shell ps | grep appname
```

---

## What to Look for in Logcat

### Crash — FATAL EXCEPTION
```
E AndroidRuntime: FATAL EXCEPTION: main
E AndroidRuntime: Process: com.phonepe.app, PID: 28988
E AndroidRuntime: java.lang.NullPointerException
E AndroidRuntime:   at com.phonepe.app.MainActivity.onCreate(MainActivity.java:45)
```
This tells you: app crashed, which process, which exception, which file and line number.

### ANR — App Not Responding
```
E ActivityManager: ANR in com.phonepe.app
E ActivityManager: Reason: Input dispatching timed out
```
App froze for 5+ seconds. User sees "App is not responding" dialog.

### OutOfMemoryError
```
E dalvikvm: Out of memory on a 4096-byte allocation
E AndroidRuntime: java.lang.OutOfMemoryError
```
App tried to allocate memory and failed. Usually causes crash.

### Garbage Collection
```
I com.phonepe.app: Background concurrent mark compact GC freed 
12MB AllocSpace bytes, paused 7.843ms total 482.588ms
```
GC pause of 482ms means app froze for half a second cleaning memory. Users experience this as a stutter or freeze.

---

## Real Findings from Today — PhonePe on LAVA LZG409

### Finding 1 — GC Pause 482ms
```
I com.phonepe.app: Background concurrent mark compact GC freed 
12MB AllocSpace bytes, 224(14MB) LOS objects, 47% free, 
26MB/50MB, paused 7.843ms,12.240ms total 482.588ms
```

**Analysis:**
- GC freed 12MB + 14MB large objects = 26MB total cleaned
- Only 47% heap free after GC — memory pressure
- Total pause: 482ms — app froze for nearly half a second
- On a payment app, a 482ms freeze during transaction confirmation is unacceptable

**Severity:** Medium — performance degradation on low-end devices

### Finding 2 — App Configuration Warning
```
W PackageConfigPersister: App-specific configuration not found 
for packageName: com.phonepe.app and userId: 0
```
Missing app-specific display configuration for this device. May cause display rendering issues on LAVA devices.

### Finding 3 — Memory Analysis (dumpsys meminfo)

| Component | Usage | Assessment |
|-----------|-------|------------|
| Native Heap | 24MB used / 31.4MB allocated | Normal |
| Dalvik Heap | 26MB used / 34.4MB allocated | Normal |
| .apk mmap | **108MB** | ⚠️ Very large |
| SwapPss Dirty | 24-27MB | ⚠️ Memory pressure |
| Total RSS | **254MB** | ⚠️ High for budget device |

**Key finding:** PhonePe uses 254MB of actual RAM on a budget LAVA device. The APK mapping alone is 108MB. Combined with swap pressure (memory pushed to storage), this explains the GC pause and stuttering behaviour.

### Finding 4 — Process Info
```
u0_a205  28988  540  12248360  259832  S  com.phonepe.app
```
- PID: 28988
- RSS: 259,832 KB = **254MB actual RAM**
- State: S (Sleeping — waiting for user input)
- Virtual memory: 12GB address space (normal for modern Android apps)

### Finding 5 — Screenshot captured via ADB ✅
```
/sdcard/screenshot.png: 1 file pulled
Size: 233,143 bytes = 227KB
Speed: 18.1 MB/s
```
Screenshot taken remotely without touching the device. Used for bug report evidence.

---

## Memory Analysis — How to Read dumpsys meminfo

| Field | Meaning |
|-------|---------|
| Pss Total | Proportional Set Size — most accurate memory measure |
| Private Dirty | Memory only this app uses — most important |
| Private Clean | Shared memory pages this app has loaded |
| SwapPss Dirty | Memory pushed to swap — indicates memory pressure |
| Heap Size | Total heap allocated to the app |
| Heap Alloc | How much heap is actually used |
| Heap Free | Available heap space |

**Rule of thumb:**
- Private Dirty under 50MB = good
- Private Dirty 50-100MB = monitor
- Private Dirty over 100MB = investigate
- SwapPss Dirty over 20MB = device under memory pressure

---

## ADB for QA — Interview Framework

**Question: "How do you investigate an Android crash?"**

"I connect the device via USB with USB debugging enabled and run `adb devices` to confirm connection. I clear existing logs with `adb logcat -c` so I only capture fresh output. I reproduce the crash scenario on the device while running `adb logcat | grep -E "FATAL|Exception"` in Terminal. When the crash occurs I see the FATAL EXCEPTION with the full stack trace — the exact class name, method, and line number. I save the full logs with `adb logcat > crash_logs.txt` and attach them to the bug report along with a screenshot taken via `adb shell screencap`. I also run `adb shell dumpsys meminfo` to check if the crash was memory-related."

---

## Key Concepts — Background vs Suspended (from Day 10, relevant here)

ADB logcat reveals the lifecycle transitions clearly:

**App going to background:**
```
I WindowManager: Input focus has changed to null from 
com.phonepe.app/com.phonepe.app.launch.core.main.ui.MainActivity
```

**App snapshot taken (suspended):**
```
D TaskStackListenerImpl: onTaskSnapshotChanged
mTopActivityComponent=com.phonepe.app/.launch.core.main.ui.MainActivity
```

These log lines confirm exactly when the app transitioned states — useful for investigating lifecycle bugs.

---

## Interview Talking Points

- Set up ADB on macOS and connected LAVA Android 14 device
- Know all log levels and when each is relevant for QA
- Found GC pause of 482ms in PhonePe — performance finding on budget device
- Identified 254MB RAM usage by PhonePe — high for budget device
- Found SwapPss pressure indicating device memory constraints
- Can read dumpsys meminfo and identify memory issues
- Can capture screenshots remotely via ADB for bug reports
- Know the 7 essential ADB commands and when to use each
- Can identify app lifecycle transitions from logcat output
- Know how to find FATAL EXCEPTION, ANR, and OutOfMemoryError in logs


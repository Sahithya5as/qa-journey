# Payment System QA Investigation — End to End
**Phase 2 Project | Network, Logs & Monitoring**
**Duration:** 7 days of investigation
**Tools used:** Chrome DevTools, Charles Proxy, ADB, Xcode Console, grep/Terminal
**Platforms:** Web, iOS (iPhone 14), Android (LAVA LZG409)
**Applications investigated:** DemoQA Bookstore API, Zomato iOS, PhonePe Android

---

## Project Overview

This project documents a complete end-to-end quality investigation of a payment and delivery system stack. The investigation covers all four layers of a modern mobile application:

| Layer | Tool used | Findings |
|-------|----------|---------|
| API / Network traffic | Chrome DevTools + Charles Proxy | 7 bugs including 2 Critical security |
| iOS application | Xcode Console + Charles | 4 bugs — SQLite error, jailbreak detection, sensor overuse |
| Android application | ADB logcat + dumpsys | 4 bugs — GC pause, memory pressure, APK size |
| Backend server | Log analysis — grep + Terminal | 5 incidents — duplicate charge, DB outage, brute force |
| Security | Charles + ADB + iOS logs | Certificate pinning verified, debug mode confirmed disabled |

**Total findings: 20 across the full stack**

---

## Investigation Methodology

### 1. API Traffic Analysis
Set up Charles Proxy on macOS with SSL decryption enabled. Configured iPhone and Android device to proxy all traffic through Charles. Investigated authentication flows, token handling, sensitive data exposure, and network performance.

### 2. Mobile Application Log Analysis
**iOS:** Connected iPhone 14 via Xcode Devices and Simulators. Captured live console logs. Filtered by application name. Identified jailbreak detection implementation, database errors, and excessive sensor usage.

**Android:** Connected LAVA LZG409 via ADB with USB debugging enabled. Ran logcat with error-level filtering. Analysed memory usage via dumpsys meminfo. Captured screenshots remotely for bug evidence.

### 3. Backend Log Investigation
Analysed payment service logs covering authentication, payment processing, database operations, and API gateway events. Identified 5 distinct production-style incidents using grep-based log analysis.

### 4. Security Testing
Verified certificate pinning on PhonePe production build. Confirmed debug mode disabled via ADB run-as. Documented jailbreak detection implementation in Zomato iOS via Sandbox deny logs. Built complete mobile security testing checklist.

---

## Key Findings Summary

### Critical Findings
| ID | Finding | Layer | Impact |
|----|---------|-------|--------|
| BUG-D001 | Password returned in plain text in API response | API | User credentials exposed |
| BUG-D002 | Password embedded in JWT token payload | API | Token decoding reveals password |
| BUG-L001 | Duplicate transaction — user charged twice | Backend | Financial — double charge |

### High Findings
| ID | Finding | Layer | Impact |
|----|---------|-------|--------|
| BUG-D003 | JWT token has no expiry — valid forever | API | Stolen token = permanent access |
| BUG-C004 | Two conflicting GPS coordinates 165km apart | iOS | Wrong delivery zone |
| BUG-A001 | GC pause 482ms in payment app | Android | App freeze during transaction |
| BUG-L005 | Database connection pool — no early warning | Backend | 5 minute payment outage |

### Security Passes (features working correctly)
| Check | Result |
|-------|--------|
| PhonePe certificate pinning | ✅ Implemented |
| PhonePe debug mode disabled | ✅ Confirmed |
| Zomato jailbreak detection | ✅ Implemented |
| PhonePe TLS 1.3 | ✅ Confirmed |

---

## Tools and Commands Reference

### Charles Proxy
```
SSL Proxying: Proxy → SSL Proxying Settings → * : 443
iOS setup: WiFi proxy → Mac IP : 8888 → install cert via AirDrop
Android setup: WiFi proxy → Mac IP : 8888 → install cert via browser
Breakpoint: Proxy → Breakpoint Settings → add endpoint
Throttling: Proxy → Throttle Settings → select preset
```

### ADB
```bash
adb devices                                    # verify connection
adb logcat *:E                                 # errors only
adb logcat | grep "com.phonepe.app"            # filter by app
adb shell dumpsys meminfo com.phonepe.app      # memory analysis
adb shell getprop ro.product.model             # device info
adb shell screencap /sdcard/s.png && adb pull  # screenshot
adb shell run-as com.package.name              # debug check
```

### Backend Log Analysis
```bash
grep "ERROR\|FATAL" app.log                    # find failures
grep "TXN003" app.log                          # trace transaction
grep "ERROR" app.log | grep -o '\[.*\]' | sort | uniq -c  # errors by service
grep -B3 -A3 "FATAL" app.log                  # context around crash
wc -l app.log                                  # total lines
```

---

## Incident Reports
See `/incident-reports/` folder for full RCA documentation of each finding.

## Bug Reports
See individual day folders for detailed bug reports with steps to reproduce, evidence, and recommendations.


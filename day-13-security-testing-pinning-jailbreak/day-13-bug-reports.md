# Day 13 — Security Findings: PhonePe + Security Testing
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** LAVA LZG409 Android 14 + iPhone 14 iOS 26.2.1
**Tools:** Charles Proxy + ADB + Xcode Console

---

## Summary

All security checks on PhonePe passed. No vulnerabilities found.
The following are PASS findings — documented as security features working correctly.

| Check | Result | Severity if Failed |
|-------|--------|-------------------|
| Certificate pinning | ✅ PASS | Critical |
| Debug mode disabled | ✅ PASS | Critical |
| TLS 1.3 in use | ✅ PASS | High |
| Jailbreak detection (Zomato iOS) | ✅ PASS | High |

---

## SECURITY PASS — Certificate Pinning Confirmed

**App:** PhonePe (com.phonepe.app)
**Platform:** Android 14 — LAVA LZG409
**Test:** Charles Proxy interception attempt

**Evidence:**
```
CONNECT https://imgstatic.phonepe.com → Failed
Status: Failed
Failure: SSL handshake with client failed — certificate_unknown
Received fatal alert: certificate_unknown
Tags: [SSL Proxying]
TLS: TLSv1.3 (TLS_AES_128_GCM_SHA256)
```

**What this means:**
PhonePe correctly implements certificate pinning. When Charles Proxy attempted to intercept traffic by presenting its own certificate, PhonePe rejected it with `certificate_unknown`. The app only accepts its own server certificate — not Charles's CA or any other certificate.

**Why this is critical for a payment app:**
Without certificate pinning, any attacker on the same network could set up a proxy and intercept all PhonePe API calls — including transaction data, auth tokens, UPI PIN transmission, and user PII. Certificate pinning prevents this attack entirely.

**Recommendation for QA team:**
Verify pinning is implemented on ALL endpoints — not just main API domains. Static asset domains (imgstatic.phonepe.com) are also pinned — good practice. Verify certificate rotation process — when server certificate expires, does the app update its pinned certificate correctly?

---

## SECURITY PASS — Debug Mode Disabled

**App:** PhonePe (com.phonepe.app)
**Platform:** Android 14 — LAVA LZG409
**Test:** ADB run-as command

**Evidence:**
```bash
$ adb shell run-as com.phonepe.app ls /data/data/com.phonepe.app/
run-as: package not debuggable: com.phonepe.app
```

**What this means:**
PhonePe's production build correctly sets `android:debuggable="false"` in AndroidManifest.xml. This prevents:
- ADB debugger attachment to the process
- `run-as` access to private app data
- Easier Frida hooking (some techniques require debuggable apps)
- Access to app's SQLite databases and SharedPreferences via ADB

**What would happen if debug was enabled:**
```bash
# On a debuggable app this would work:
$ adb shell run-as com.phonepe.app ls /data/data/com.phonepe.app/
databases/
shared_prefs/
cache/
files/
# Auth tokens, user data, transaction history all readable
```
A debuggable production payment app would be a Critical security vulnerability.

---

## WHAT TO TEST ON YOUR OWN COMPANY'S DEBUG BUILD

When working as a QA engineer at a fintech company, you will receive debug builds. These are the tests to run with Charles Proxy enabled:

**API Security Tests:**
- Auth token format and location (header vs URL)
- Token expiry time
- Sensitive data in response bodies (card numbers, CVV, OTP)
- Sensitive data in request payloads
- Error messages exposing internal paths or stack traces

**Payment Flow Tests:**
- Transaction amount in request payload — can it be modified via Charles breakpoint?
- OTP transmission — is it encrypted beyond HTTPS?
- UPI PIN — never sent to server — only processed locally?
- Duplicate transaction prevention — double submit test

**Session Security Tests:**
- Token validity after logout
- Token validity after password change
- Concurrent session handling
- Session timeout after inactivity


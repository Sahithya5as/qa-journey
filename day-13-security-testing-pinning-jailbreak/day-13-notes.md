# Day 13 — Security Testing: Jailbreak, Root Detection, Certificate Pinning
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** macOS + iPhone 14 (iOS 26.2.1) + LAVA LZG409 (Android 14)
**Tools:** Charles Proxy + ADB
**Application:** PhonePe (com.phonepe.app)

---

## 1. CA Certificates — From Scratch

### What is a Certificate
A digital certificate is like a passport for a website or server. It proves:
- Who the server is (e.g. api.phonepe.com)
- That a trusted authority verified its identity
- That it has not expired

### What is a CA (Certificate Authority)
A CA is like the government that issues passports — a trusted organisation that verifies and signs certificates. Well-known CAs include DigiCert, Let's Encrypt, Comodo, and GlobalSign. Every iPhone and Android comes with a pre-installed list of trusted CAs.

### How HTTPS Works
```
App connects to server
        ↓
Server sends its certificate: "I am api.phonepe.com, verified by DigiCert"
        ↓
Phone checks: Is DigiCert on my trusted CA list? → YES
Phone checks: Has this certificate expired? → NO
        ↓
Encrypted connection established
```

---

## 2. How Charles Proxy Intercepts HTTPS

Charles acts as a man-in-the-middle between the app and the server.

```
Normal:   App ←→ Server
Charles:  App ←→ CHARLES ←→ Server
```

Charles installs its own CA certificate on the device. Once trusted, Charles presents its own certificate for any domain. The device checks its trusted CA list, finds Charles CA, and accepts it. Charles decrypts and re-encrypts all traffic — seeing everything.

**This is why installing the Charles certificate was required on Days 9 (iOS and Android).**

---

## 3. Certificate Pinning — What It Is and Why It Matters

### Definition
Certificate pinning means an app ignores the device's trusted CA list entirely and only accepts one specific certificate — its own server's certificate hardcoded inside the app.

```
Normal app:   "Is this cert from a trusted CA?" → Yes → ACCEPT
Pinned app:   "Is this EXACTLY our server cert?" → No (Charles) → REJECT
```

### Real World Analogy
A normal club accepts any valid government ID. A pinned club only accepts one specific membership card with one specific photo. Charles is like someone entering with a well-made fake ID — the pinned club rejects it because it only accepts one card.

### Why Fintech Apps Use Certificate Pinning

| Risk without pinning | Impact |
|---------------------|--------|
| Traffic interception | Transaction details exposed |
| Token theft | Auth tokens readable |
| Request modification | Payment amounts changeable |
| Credential theft | Login details extractable |

### What Charles Shows When Pinning Is Active
```
Status: Failed
Failure: SSL handshake with client failed — certificate_unknown
Notes: You may need to configure your browser or 
       application to trust the Charles Root Certificate
```
Left panel shows `<unknown>` with red X icons.

**Confirmed on PhonePe today:**
```
URL: imgstatic.phonepe.com
Status: Failed
Failure: SSL handshake with client failed — certificate_unknown
Client: 192.168.1.5 (Android phone)
Tags: [SSL Proxying]
```

---

## 4. Jailbreak (iOS) vs Rooting (Android)

### iOS Jailbreaking
Exploiting a vulnerability to remove Apple's sandbox restrictions. Gives root access to the entire file system.

**What becomes possible:**
- Install unapproved apps via Cydia
- Access any app's private data
- Bypass certificate pinning using SSL Kill Switch
- Modify app binaries at runtime
- Read auth tokens from memory

**Common jailbreak indicators:**
```
/bin/bash                    — Unix shell
/private/etc/apt             — Cydia package manager
/Applications/Cydia.app      — Cydia itself
/Library/MobileSubstrate     — jailbreak framework
```

**Confirmed in Zomato iOS logs (Day 12):**
```
kernel: Sandbox: Zomato(677) deny(2) file-test-existence /bin/bash
kernel: Sandbox: Zomato(677) deny(2) file-test-existence /private/etc/apt
```
Zomato checks for these files on every launch. iOS sandbox blocks and logs the checks.

### Android Rooting
The Android equivalent of jailbreaking. Gaining superuser (root) access.

**Common root indicators:**
```
/sbin/su                      — su binary
/system/bin/su                — su binary location
/system/xbin/su               — another location
com.noshufou.android.su       — SuperUser app
eu.chainfire.supersu          — SuperSU app
```

### Why the Jailbreak/Root → Pinning Bypass Chain Matters
```
Jailbroken/Rooted device
        ↓
Frida or SSL Kill Switch can be installed
        ↓
These tools hook into the app at runtime
        ↓
Certificate pinning check is bypassed
        ↓
Charles can now intercept ALL traffic
        ↓
Auth tokens, transactions, PII all exposed
```
**This is why fintech apps check for jailbreak/root — to break this chain.**

---

## 5. PhonePe Security Findings Today

### Finding 1 — Debug Mode Disabled ✅ PASS
```bash
adb shell run-as com.phonepe.app ls /data/data/com.phonepe.app/
# Output: run-as: package not debuggable: com.phonepe.app
```

**What this means:**
PhonePe's production build has `android:debuggable="false"` in AndroidManifest.xml. This means:
- No one can use `run-as` to access PhonePe's private data
- ADB cannot attach a debugger to the process
- Frida and other tools are significantly harder to use
- Private databases, tokens, and user data are protected

**If this had been debuggable:** Anyone with ADB access to the device could read PhonePe's SQLite databases, SharedPreferences, auth tokens, and user data — Critical security vulnerability.

### Finding 2 — Certificate Pinning Implemented ✅ PASS
```
Charles result on PhonePe Android traffic:
Status: Failed
Failure: SSL handshake with client failed — certificate_unknown
Domains affected: apicp1.phonepe.com, imgstatic.phonepe.com
```

Charles cannot decrypt any PhonePe traffic. All domains show `<unknown>` with red X. Certificate pinning is correctly implemented across all PhonePe endpoints.

### Finding 3 — Production Build Confirmed ✅ PASS
No debug flags found in package dump. App is running as a production build with full security measures active.

---

## 6. Security Testing Checklist for Mobile QA

### Authentication Security
- [ ] Token stored in secure storage (Keychain iOS / Keystore Android)
- [ ] Token not in UserDefaults/SharedPreferences unencrypted
- [ ] Token not visible in logs
- [ ] Token not in URL parameters
- [ ] Token expires in 15-60 minutes (access token)
- [ ] Account locks after N failed attempts

### Data Security
- [ ] Sensitive data not in logs (password, card number, OTP)
- [ ] Screenshot prevention on sensitive screens
- [ ] Data encrypted at rest
- [ ] No sensitive data in unencrypted local storage

### Network Security
- [ ] Certificate pinning implemented
- [ ] All traffic over HTTPS — no HTTP
- [ ] No sensitive data in URL parameters
- [ ] Proper SSL/TLS version (TLS 1.2 minimum, TLS 1.3 preferred)

### Device Security
- [ ] Jailbreak detection implemented (iOS)
- [ ] Root detection implemented (Android)
- [ ] Debug mode disabled in production build
- [ ] App blocks critical features on compromised devices

### Privacy
- [ ] Only necessary permissions requested contextually
- [ ] Location precision appropriate
- [ ] No excessive sensor usage
- [ ] ATT implemented on iOS
- [ ] Analytics not collecting PII without consent

---

## 7. How to Test These in Real QA Work

### Testing certificate pinning
Set up Charles Proxy → proxy device traffic → open app → if `<unknown>` with SSL error → pinning confirmed.

### Testing jailbreak/root detection
Use a dedicated lab device that is jailbroken/rooted. Test that app shows warning and blocks critical features.

### Testing debug mode
```bash
adb shell run-as com.package.name ls
# If "package not debuggable" → PASS
# If file list appears → FAIL — Critical bug
```

### Testing in your own company
Your company will give you a **debug build** — a special version of the app where pinning is disabled and debug mode is on. You use Charles on this build to test API calls. The production build always has pinning enabled.

---

## Interview Talking Points

- Can explain CA certificates, HTTPS, and how Charles intercepts traffic — using simple analogies
- Know what certificate pinning is and why fintech apps use it
- Confirmed PhonePe has certificate pinning via Charles SSL handshake failure
- Confirmed PhonePe debug mode is disabled via ADB run-as command
- Know the difference between iOS jailbreaking and Android rooting
- Know the jailbreak → pinning bypass chain and why it matters
- Have seen jailbreak detection in real app logs (Zomato Day 12)
- Know the complete mobile security testing checklist
- Understand that QA uses debug builds internally — not production builds with pinning


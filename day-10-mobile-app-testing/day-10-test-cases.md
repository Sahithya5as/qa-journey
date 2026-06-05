# Day 10 — Test Cases: Mobile App Testing
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** iPhone iOS 26.2.1 + Android
**Application:** Zomato iOS
**Focus:** Installation, lifecycle, notifications, deep links, network conditions

---

## Execution Summary

| Total | Passed | Failed | Observations |
|-------|--------|--------|--------------|
| 28 | 18 | 5 | 5 |

---

## Section 1 — Installation Testing

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_M_001 | App installs successfully from App Store | Clean install, no errors | Installed in 30 seconds | ✅ Pass |
| TC_M_002 | Location permission requested contextually | Asked when user first searches, not on launch | Asked BEFORE login screen | ⚠️ Fail |
| TC_M_003 | Permissions requested one at a time contextually | Each permission asked when that feature is used | Multiple permissions asked upfront before login | ⚠️ Fail |
| TC_M_004 | Login cleared after complete uninstall | User must re-authenticate after reinstall | OTP required — login correctly cleared | ✅ Pass |
| TC_M_005 | Onboarding shown on fresh install | First-time experience guide shown | Onboarding shown | ✅ Pass |
| TC_M_006 | App does not crash on first launch | Stable launch on fresh install | Launched successfully | ✅ Pass |

---

## Section 2 — Foreground and Background Testing

| TC ID | Scenario | Steps | Expected | Actual | Status |
|-------|----------|-------|----------|--------|--------|
| TC_M_007 | App resumes correct page after 2 min background | Open app → Home → wait 2 min → return | Same page shown | Same page shown — no reload | ✅ Pass |
| TC_M_008 | App refreshes data after background resume | Open app → Home → 2 min → return → check Charles | API call fires within 5 seconds of resume | No API call observed — cached data shown | 👁️ Observation |
| TC_M_009 | App handles phone call interruption | Use app → receive call → end call → return | Data preserved, no crash | App behaved normally — data preserved | ✅ Pass |
| TC_M_010 | App handles multi-app switching | Open app → switch to 3 apps → return | App on same page, no data loss | Same page, data intact | ✅ Pass |
| TC_M_011 | Cart preserved after background | Add item to cart → background → foreground | Cart items preserved | ✅ Pass | ✅ Pass |

---

## Section 3 — Push Notification Testing

| TC ID | Scenario | Expected | Actual | Status |
|-------|----------|----------|--------|--------|
| TC_M_012 | Notification shown when app in foreground | Banner shown inside app OR system notification | No notification shown — silently dropped | ⚠️ Fail |
| TC_M_013 | Notification shown when app in background | System notification in notification centre | Notification appeared with correct content | ✅ Pass |
| TC_M_014 | Background notification content correct | Shows right restaurant/order info | Correct restaurant and order info shown | ✅ Pass |
| TC_M_015 | Background notification navigates to correct screen | Tapping opens exact screen referenced | Opened correct screen directly | ✅ Pass |
| TC_M_016 | Notification received when app is killed | System notification delivered | Notification received after force close | ✅ Pass |
| TC_M_017 | Killed app notification navigation | Opens correct screen after cold start | Home screen briefly shown then correct screen | ✅ Pass (acceptable) |
| TC_M_018 | Badge count accurate | Shows correct unread count | Correct count shown | ✅ Pass |
| TC_M_019 | Badge count clears after opening app | Count goes to 0 after viewing | Count cleared correctly | ✅ Pass |

---

## Section 4 — Deep Link Testing

| TC ID | Scenario | Steps | Expected | Actual | Status |
|-------|----------|-------|----------|--------|--------|
| TC_M_020 | Universal link from Safari browser | Open restaurant in Safari → tap Open in App | App opens to correct restaurant | App opened correct restaurant page | ✅ Pass |
| TC_M_021 | Deep link from WhatsApp | Tap restaurant link in WhatsApp | App opens to correct screen | Not tested | 🚫 Blocked |
| TC_M_022 | Deferred deep link when app not installed | Tap link with app uninstalled | Redirected to App Store | Redirected to App Store correctly | ✅ Pass |
| TC_M_023 | After install from deferred deep link | Install from App Store via deep link | Opens correct page after install | Correct page opened | ✅ Pass |

---

## Section 5 — Network Condition Testing

| TC ID | Scenario | Steps | Expected | Actual | Status |
|-------|----------|-------|----------|--------|--------|
| TC_M_024 | Airplane mode — error message shown | Enable airplane mode → open app | Clear no internet message | "No internet connection" shown | ✅ Pass |
| TC_M_025 | Airplane mode — cached content shown | Enable airplane mode → open app | Last known restaurant list shown | Blank error screen — no cached content | ⚠️ Fail |
| TC_M_026 | Airplane mode — app does not crash | Enable airplane mode → open app | No crash | App stable — no crash | ✅ Pass |
| TC_M_027 | Airplane mode — retry button present | Enable airplane mode → open app | Retry button visible | Retry button shown | ✅ Pass |
| TC_M_028 | WiFi to mobile data switch | Use app on WiFi → disable WiFi | Seamless continuation | App continued working seamlessly | ✅ Pass |


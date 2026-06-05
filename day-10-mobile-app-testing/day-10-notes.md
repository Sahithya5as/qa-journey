# Day 10 — Mobile App Testing Deep Dive
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** iPhone + Android — real devices
**Application Tested:** Zomato iOS (fresh install + full lifecycle testing)

---

## Mobile App Testing — Why It Is Different from Web Testing

Web testing happens in a controlled browser environment. Mobile app testing introduces an entirely new set of variables — device state, OS behaviour, network conditions, hardware interactions, and app lifecycle management. A bug that never appears in a browser can appear every time on a mobile device under specific conditions.

---

## 1. App Lifecycle — Complete Guide

### The 5 States

| State | Description | Code executing? | In memory? |
|-------|-------------|-----------------|------------|
| Not Running | App not launched or was killed | ❌ No | ❌ No |
| Foreground | App visible on screen, user interacting | ✅ Yes | ✅ Yes |
| Background | App minimised but still running | ✅ Yes | ✅ Yes |
| Suspended | OS froze the app — no code running | ❌ No | ✅ Yes |
| Terminated | Killed by user or OS needing memory | ❌ No | ❌ No |

### Background vs Suspended — Key Distinction

**Background:** App is alive and working — just not visible. It can make API calls, track location, play audio, process data. Example: Spotify playing music while you use WhatsApp. Example: food delivery app tracking your order location after you switch apps.

**Suspended:** App is frozen in memory — like a photograph. No code executes. The OS can kill it at any time if memory is needed. Example: Calculator app after you switch to Instagram — still in memory, your last result visible, but doing nothing.

```
Foreground → Home button → Background → OS freezes → Suspended → OS needs memory → Not Running
```

### Cold Start vs Warm Start vs Hot Start

| Type | Condition | Speed | What to test |
|------|-----------|-------|-------------|
| Cold start | App not in memory | Slowest | Launch time, crash on first load |
| Warm start | App was suspended | Medium | State restoration, data freshness |
| Hot start | App just minimised | Fastest | Immediate resume, no data loss |

---

## 2. Installation Testing — Zomato iOS

**App:** Zomato
**Install time:** 30 seconds from App Store
**Test device:** iPhone, iOS 26.2.1

### Permission Request Analysis

**Finding — Premature location permission request:**
Location permission was requested BEFORE the login screen appeared. This is a UX anti-pattern. Best practice is to request permissions contextually — ask for location only when the user first tries to search for nearby restaurants, not on first app launch.

| Permission | When requested | Expected | Actual |
|-----------|----------------|----------|--------|
| Location | Before login | After login, when needed | Before login — premature |
| Access to other apps | Before login | When feature is used | Before login — premature |

**Security finding — Login cleared after reinstall:**
After complete uninstall and reinstall, the user was required to enter phone number and OTP again. Login session was not persisted. This is correct security behaviour — authentication tokens should be cleared on uninstall.

**Interview talking point:** "I verify that sensitive session data is cleared on app uninstall. If a user uninstalls and reinstalls and is still logged in — that is a security vulnerability because someone else could install the app on the same device and access the previous user's account."

---

## 3. Foreground and Background Testing

### Test 1 — 2 Minute Background Resume
**Action:** Opened Zomato home screen → pressed Home → waited 2 minutes → returned to app
**Result:** App resumed on exact same page with no loading spinner

**Analysis — Should Charles show API calls on resume?**
It depends on app design. Three possible behaviours:

| Behaviour | What it means | Good or bad? |
|-----------|--------------|--------------|
| No API call, same content shown | Showing cached data | ⚠️ Risk of stale data |
| Silent API call, content updates | Background refresh working | ✅ Best practice |
| Spinner shown, fresh API call | Full refresh on resume | ✅ Acceptable |

For a food delivery app where restaurant availability and prices change constantly, showing 2-minute-old cached data without a refresh is a potential bug. A user might see a restaurant as open when it has just closed.

**What to check in Charles:** After resuming from background, watch Charles for 5 seconds. If no API call fires but content appears — the app is showing cached data. Document the cache duration.

### Test 2 — Phone Call Interruption
**Action:** Used app normally → received phone call → answered → ended call → returned to app
**Result:** App behaved normally — data preserved, no crash, no restart

### Test 3 — Multi-app Switching
App correctly maintained state when switching between multiple apps and returning.

---

## 4. Push Notification Testing

### Test 1 — Foreground Notification
**Finding:** No notification shown when app was in foreground.

**Why this happens on iOS:**
iOS does not automatically show push notifications when the app is in the foreground. The app must explicitly implement `UNUserNotificationCenterDelegate` and call `completionHandler` with display options to show foreground notifications. If the developer has not implemented this — notifications are silently dropped when the app is active.

**QA implication:** This is a functional bug. If the app is open and an order status changes, the user gets no visual alert. They only see it if they go background first.

**Bug raised:** No foreground push notification displayed when app is active.

### Test 2 — Background Notification
**Result:** Notification appeared in notification centre with correct content (right restaurant, right order details). Tapping opened the correct screen inside the app. ✅

### Test 3 — Killed App Notification
**Result:** Notification received when app was force closed. Tapping notification:
1. App launched (cold start)
2. Home screen shown briefly
3. Then navigated to the correct screen

**Analysis:** The brief home screen flash before deep link navigation is acceptable behaviour during a cold start. However the final destination was correct — deep link from notification worked. ✅

### Test 4 — Badge Count
Badge count on app icon showed correct unread count. Count cleared after opening app. ✅

---

## 5. Deep Link Testing

### Types of Deep Links

| Type | Format | Example |
|------|--------|---------|
| URI Scheme | `myapp://path` | `zomato://restaurant/12345` |
| Universal Link (iOS) | `https://domain.com/path` | `https://zomato.com/restaurant/xyz` |
| App Link (Android) | `https://domain.com/path` | Same as above but Android |
| Deferred Deep Link | Any | Redirects to App Store if not installed |

### Test Results

**Browser to app deep link:**
Opened Zomato restaurant page in Safari → tapped "Open in app" banner → app opened and navigated directly to the correct restaurant page. ✅

**Deep link when app not installed (deferred deep link):**
Tapped a Zomato link with app uninstalled → redirected to App Store to install. After install, app opened to the correct page. ✅ This is a well-implemented deferred deep link.

**What makes a good deep link implementation:**
- Correct screen opens — not just the home screen
- Works when app is in all states — foreground, background, killed
- Works when app is not installed — redirects to store
- After installing from deferred deep link — correct screen opens

---

## 6. Network Condition Testing

### Test 1 — Airplane Mode (No Internet)
**Action:** Enabled Airplane mode → opened Zomato
**Results:**
- ✅ Clear "No internet connection" error message shown
- ❌ No cached content displayed — blank error screen (finding)
- ✅ App did not crash
- ✅ Retry button present

**Bug found:** App shows blank error screen with no cached content when offline. Best practice for food delivery apps is to show the last known restaurant list from cache so users can browse even without internet, with a banner indicating the content may be outdated.

### Test 2 — Throttled Network (Charles ADSL2)
Tested app behaviour on simulated slow network.
Skeleton loading screens appeared ✅
Images loaded progressively ✅
No timeouts or crashes observed ✅

### Test 3 — Network Switch (WiFi to Mobile Data)
App continued working seamlessly when WiFi was disabled and mobile data took over. No errors or interruptions observed. ✅

---

## 7. Background vs Suspended — Interview Explanation

**Full answer for interviews:**

"Background and suspended are two different iOS app states that QA engineers must test explicitly. When an app goes to background — the user presses Home — it is still running. It can make network requests, track location, play audio if it has the right background mode permissions. Suspended means the OS has frozen the app — it is in memory but executing no code at all. The OS can terminate it at any time.

The bugs I look for at these transitions are: does the app show stale data when resuming from suspended state? Does it make a refresh API call or serve cache? Does it lose cart items or form data? On food delivery apps, showing prices from 30 minutes ago without a refresh is a real business bug — the user might order at the wrong price.

I use Charles Proxy to watch for API calls during resume — if no call fires and content appears immediately, I know the app is showing cached data and I document the cache duration as a finding."

---

## 8. Android-Specific Considerations

### Doze Mode
When an Android phone is idle and on battery, Doze mode restricts background app activity. API calls, location updates, and syncs are deferred.
**QA test:** Leave phone idle for 30 minutes → check if app still receives push notifications.

### App Standby
Rarely used apps are put into standby — background activity restricted.
**QA test:** Clear an app from recents, do not use it for 24 hours, check if push notifications still arrive.

### Battery Optimisation Setting
Users can restrict specific apps under Settings → Battery → Battery Optimisation.
**QA test:** Set app to "Optimised" → test if push notifications and background refresh still work.

---

## 9. Tools for Mobile Log Analysis

### Android — ADB Logcat
```bash
# All logs from device
adb logcat

# Only error level logs
adb logcat *:E

# Filter by app package name
adb logcat | grep "com.application.package"

# Save logs to file
adb logcat > device_logs.txt

# Clear existing logs first
adb logcat -c
```

**What to look for:**
- `FATAL EXCEPTION` — app crash
- `ANR` — App Not Responding — app froze for 5+ seconds
- `OutOfMemoryError` — memory issue
- `NullPointerException` — common crash cause

### iOS — Xcode Console
1. Connect iPhone via cable
2. Open Xcode → Window → Devices and Simulators
3. Select your iPhone
4. Click **Open Console**
5. Filter by app name in search box

**What to look for:**
- `fault:` prefix — crash level
- `error:` prefix — error level
- Stack traces after a crash

---

## Interview Talking Points

- Can explain all 5 app lifecycle states and the difference between background and suspended
- Know cold start vs warm start vs hot start and what to test at each transition
- Found premature permission request bug — location asked before login
- Verified correct security behaviour — login cleared on reinstall
- Found foreground push notification bug — iOS requires explicit implementation
- Know the difference between URI scheme deep links and universal links
- Found offline caching gap — app shows blank error instead of cached content
- Know Android Doze mode, App Standby, and battery optimisation effects on testing
- Can read ADB logcat and filter for crashes, ANRs, and errors
- Can use Xcode console for iOS log analysis


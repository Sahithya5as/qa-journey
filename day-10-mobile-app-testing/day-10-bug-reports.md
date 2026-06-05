# Day 10 — Bug Reports: Mobile App Testing
**Phase:** 2 — Network, Logs & Monitoring
**Environment:** iPhone iOS 26.2.1
**Application:** Zomato iOS
**Found using:** Manual testing — installation, lifecycle, notifications, network

---

## Summary

| Bug ID | Title | Severity | Priority | Found In |
|--------|-------|----------|----------|----------|
| BUG-M001 | Location permission requested before login screen — premature permission request | Medium | Medium | Installation |
| BUG-M002 | No push notification shown when app is in foreground — silently dropped | High | High | Notifications |
| BUG-M003 | No cached content shown in offline mode — blank error screen | Medium | Medium | Network |
| BUG-M004 | No data refresh API call after resuming from 2-minute background — stale data risk | Medium | Medium | Lifecycle |
| BUG-M005 | Multiple permissions requested upfront before login — UX anti-pattern | Low | Medium | Installation |

---

## BUG-M001

**Title:** Zomato iOS — Location permission dialog shown before login screen on fresh install — premature permission request — iPhone iOS 26.2.1

**Severity:** Medium
**Priority:** Medium
**Type:** UX — Permission Management
**Found by:** Installation testing — fresh install from App Store

**Steps to Reproduce:**
1. Uninstall Zomato completely from iPhone
2. Install Zomato fresh from App Store
3. Open the app for the first time
4. Observe when location permission dialog appears

**Expected Result:**
Location permission should be requested contextually — only when the user first tries to search for nearby restaurants or set a delivery location. The user should see the login screen first so they understand the app before being asked for permissions.

**Actual Result:**
Location permission dialog appears before the login screen. The user is asked to share their location before they have even seen the app or agreed to use it.

**Impact:**
Medium — users who see permission requests before understanding the app context are more likely to deny them. Research shows contextual permission requests have significantly higher acceptance rates than upfront requests. Denied location permission leads to poor user experience in a location-dependent app like Zomato.

**Best Practice:**
Apple's Human Interface Guidelines explicitly recommend requesting permissions only when your app clearly needs it to complete a task the user is trying to do.

---

## BUG-M002

**Title:** Zomato iOS — Push notification not displayed when app is in foreground — notification silently dropped — iPhone iOS 26.2.1

**Severity:** High
**Priority:** High
**Type:** Functional — Push Notification
**Found by:** Push notification testing — foreground state

**Steps to Reproduce:**
1. Open Zomato app and keep it in foreground
2. Trigger or wait for a push notification (order update, promotional)
3. Observe — no notification banner appears

**Expected Result:**
When a push notification arrives while the app is in foreground, it should display as either:
- An in-app banner at the top of the screen
- A system notification banner
- An in-app alert or badge update

**Actual Result:**
No notification is shown at all when the app is in foreground. The notification is silently dropped.

**Technical Root Cause:**
iOS requires apps to explicitly implement the `userNotificationCenter(_:willPresent:withCompletionHandler:)` delegate method to show notifications in the foreground. If this is not implemented, iOS silently drops foreground notifications. The developer has not implemented this delegate method.

**Impact:**
High — if a user has the Zomato app open while waiting for their order, they receive no notification when the order status changes. They must manually check the order screen. In a food delivery context, missing order status notifications (out for delivery, delivered) is a significant UX failure.

---

## BUG-M003

**Title:** Zomato iOS — No cached content shown when device is offline — blank error screen displayed instead of last known restaurant list — iPhone iOS 26.2.1

**Severity:** Medium
**Priority:** Medium
**Type:** Functional — Offline Behaviour
**Found by:** Network testing — Airplane mode

**Steps to Reproduce:**
1. Open Zomato and browse restaurants normally (so cache is populated)
2. Enable Airplane mode
3. Reopen Zomato or pull to refresh

**Expected Result:**
App should display the last known restaurant list from local cache with a banner: "You are offline. Showing saved content. Pull to refresh when connected."

**Actual Result:**
Blank error screen with "No internet connection" message. No cached content shown despite the app having loaded restaurants previously in the same session.

**Impact:**
Medium — users in areas with intermittent connectivity (common in India) frequently lose and regain network. Showing a blank screen instead of cached content creates a poor experience. Competitors like Swiggy show cached restaurant lists offline. For a fintech app like Barq, showing cached transaction history offline is even more important.

**Best Practice:**
Cache restaurant list, search results, and order history locally. Show cached data with a clear offline indicator. Only show error for actions that require real-time data (placing an order, checking live prices).

---

## BUG-M004

**Title:** Zomato iOS — No data refresh API call observed after app resumes from 2-minute background — potential stale data displayed — iPhone iOS 26.2.1

**Severity:** Medium
**Priority:** Medium
**Type:** Functional — Data Freshness
**Found by:** Background/foreground lifecycle testing — verified via Charles Proxy

**Steps to Reproduce:**
1. Open Zomato — load the home restaurant list
2. Press Home button — app goes to background
3. Wait 2 minutes
4. Return to Zomato
5. Monitor Charles Proxy for API calls in the 5 seconds after resume

**Expected Result:**
Within 5 seconds of resuming from background, the app should fire a refresh API call to check for updated restaurant availability, pricing, and offers.

**Actual Result:**
No API call observed in Charles Proxy after resuming. The exact same content from 2 minutes ago is displayed without any refresh.

**Impact:**
Medium — restaurant availability changes frequently. A restaurant open 2 minutes ago may now be closed. Prices and offers change. A user tapping a restaurant shown as "Open" might find it is actually closed when they try to place an order — causing frustration and order abandonment. 2-minute stale data is acceptable for many apps but not for a real-time food delivery service.

**Suggested Fix:**
Implement a background refresh that fires when the app returns to foreground. Alternatively, check data freshness timestamp and refresh if content is older than 60 seconds.

---

## BUG-M005

**Title:** Zomato iOS — Multiple permission requests shown upfront before login — violates contextual permission best practice — iPhone iOS 26.2.1

**Severity:** Low
**Priority:** Medium
**Type:** UX — Permission Management
**Found by:** Installation testing

**Steps to Reproduce:**
1. Fresh install Zomato
2. Open app for the first time
3. Observe multiple permission dialogs before login screen

**Expected Result:**
No permissions should be requested before login. After login, permissions should be requested one at a time only when the relevant feature is first used.

**Actual Result:**
Both location permission AND access to other apps permission requested before login screen is shown.

**Impact:**
Low — users who deny permissions before understanding the app are harder to re-engage. Apple guidelines recommend contextual permission requests. Multiple upfront requests feel intrusive and reduce trust.


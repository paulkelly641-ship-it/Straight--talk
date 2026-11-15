# Build & Upload Steps — Straight Talk AI for iOS

Modeled on the working My Baby Smarter / Potty Mouth flow (Codemagic +
GitHub, no Mac). Do the steps in order. Steps 1–4 can be done in Safari
on a phone.

## 0. What you need first

- This project pushed to a GitHub repo (e.g. `straight-talk-app`).
- An Apple Developer account (already have it — Potty Mouth and
  My Baby Smarter are on it).
- Codemagic connected to the repo, with the **`appstore` environment
  group** already configured from Potty Mouth (App Store Connect API
  key + certificate private key). If Potty Mouth builds in Codemagic,
  nothing new is needed here.
- A free RevenueCat account (revenuecat.com).
- Your own OpenAI API key is **optional** and is never part of the
  build — users add their own key in the app's Settings.

## 1. Create the app in App Store Connect (phone OK)

1. Open [App Store Connect](https://appstoreconnect.apple.com) in Safari.
2. My Apps → **+** → New App.
3. Name: **Straight Talk AI**. Bundle ID: `com.paulkelly.straighttalkai`.
   (Register the bundle ID first under Certificates, Identifiers &
   Profiles if it asks.)
4. SKU: anything, e.g. `straighttalkai001`. User Access: Full Access.

## 2. Create the 3 In-App Purchase products (phone OK)

App Store Connect → Straight Talk AI → Monetization → In-App Purchases.

1. **Monthly** — Product ID `com.paulkelly.straighttalkai.monthly`, $9.99,
   **Auto-Renewable Subscription**. Subscription group: create
   **"Straight Talk Premium"**.
2. **Yearly** — Product ID `com.paulkelly.straighttalkai.yearly`, $59.99,
   **Auto-Renewable Subscription**. Same group, "Straight Talk Premium".
3. **Lifetime** — Product ID `com.paulkelly.straighttalkai.lifetime`, $99.99,
   **Non-Consumable**. (Not a subscription — do not put it in the group.)

Fill in a display name and description for each (e.g. "Straight Talk
Monthly — unlimited straight talk from Duke"). Submit them for review
with the app.

## 3. Set up RevenueCat (phone OK for the dashboard)

1. Sign up at revenuecat.com (free) and add the Straight Talk AI app (iOS,
   bundle ID `com.paulkelly.straighttalkai`).
2. Link your App Store Connect account in RevenueCat (uses the same
   App Store Connect API key — it imports your products automatically).
3. Create an **Entitlement** named exactly **`premium`** and attach all
   three products to it.
4. Create an **Offering** (the default one is fine) containing three
   packages, one per product: monthly, yearly, lifetime.
5. Copy the **public Apple API key** (starts with `appl_`) and paste it
   into `www/index.html`, replacing
   `TODO_REPLACE_WITH_YOUR_REVENUECAT_PUBLIC_APPLE_API_KEY`.
   Commit and push the change. (Public key only — never a secret key.)

## 4. Paste the store listing (phone OK)

In App Store Connect → Straight Talk AI → App Store listing, paste from
`store/APP_STORE_LISTING.md`: name "Straight Talk AI — Duke", subtitle,
promo text, description, keywords. Category: **Lifestyle**
(Entertainment). Age rating: **17+** (Frequent/Intense Profanity).

- **Icon:** upload `store/app-icon-1024.png` (1024×1024).
- **Screenshots:** 6.9" (1290×2796) and 6.5" (1242×2688) iPhone, at
  least 3 each — chat with Duke, Roast Me, paywall, Settings (API key).
  Take them in the iOS Simulator or on a real iPhone running the
  TestFlight build.
- **Privacy Policy URL:** host `store/privacy-policy.html` on GitHub
  Pages (repo Settings → Pages → deploy from the `store` folder or a
  `docs` copy) and paste that URL. Use the same URL as the Support URL
  if you have no support site. Support email: paulkelly641@gmail.com.
- **App Privacy:** no data collected (the app collects nothing; chats
  and the API key stay on-device). Answer the questionnaire accordingly.

## 5. Bank & tax info (phone OK)

App Store Connect → Agreements, Tax, and Banking → add your bank
account and tax forms. Apple cannot pay you without this. (Same step
as Potty Mouth.)

## 6. Build the IPA in Codemagic (phone OK — it runs in the cloud)

1. In Codemagic, open the repo and run the **ios-workflow**.
2. What it does (see `codemagic.yaml`): `npm ci`, writes
   `capacitor.config.json`, `npx cap add ios` + `npx cap sync ios`
   (builds the native Xcode project from `www/`), bumps the build
   number, fetches signing files from App Store Connect, builds the
   IPA, and uploads it to App Store Connect.
3. If the build fails on a Swift version error, open
   `ios/App/App.xcodeproj` (only if you have Mac access) and set
   **Swift Language Version = 5.0** in the App target — the RevenueCat
   plugin needs it when Xcode does not set it already. Then push and
   re-run. (Capacitor 8 projects normally set this already.)

## 7. Submit for Review (phone OK)

1. App Store Connect → Straight Talk AI → select the uploaded build.
2. Paste the **review notes** from `store/APP_STORE_LISTING.md`.
3. Make sure the IAP products are attached and submitted.
4. **Submit for Review.**

## 8. Test before you submit (recommended)

- Install the build via **TestFlight** on a real iPhone.
- Test with a **sandbox Apple ID**: send 5 messages (free limit),
  confirm the paywall appears on the 6th, subscribe monthly, check
  the app unlocks, kill and reopen the app (it should stay unlocked),
  restore purchases, then cancel in Settings to confirm the paywall
  returns.
- Test the 18+ gate (confirm 18+ enters; under-18 is blocked).
- Test Real AI: add your own OpenAI key in Settings, send a message,
  confirm the badge says REAL AI — and remove the key afterwards.
- Test Duke's voice: his replies should play out loud automatically;
  tap Hear it on any reply to replay it, and tap the 🎤 button to
  dictate a message (on iPhone the mic depends on the device/browser
  and can be limited — typing always works).

## Local build notes (if you ever get Mac access)

```bash
npm install
npx cap add ios
npx capacitor-assets generate --ios   # needs icon + splash in assets/
npx cap sync ios
npx cap open ios
```

In Xcode: enable automatic signing, set
**ITSAppUsesNonExemptEncryption = NO** (the app uses no encryption),
version 1.0 build 1, then Product → Archive → Distribute →
App Store Connect → Upload.

## Review tips (what Apple cares about for this app)

- **Guideline 4.2 (Minimum Functionality):** the app is not just a
  website in a wrapper — the chat, topics, starters, history, built-in
  engine, and paywall are all built in and work offline, with no
  external links. Keep it that way.
- **Guideline 3.1.1 (In-App Purchase):** all payments go through Apple
  In-App Purchase via RevenueCat/StoreKit. There is no card form and no
  external payment link anywhere in the app.
- **Guideline 4.3(a) (Design — Spam):** this is the one that got Potty
  Mouth rejected. This app is built the same wrapped-web way, so Apple
  could push back the same way. What helps here: the concept is
  genuinely different (an interactive advice chat with two reply
  engines), not a reskinned template. If Apple rejects it, do NOT
  resubmit the same app — read the reason and fix what they name.
- **Age rating 17+:** the app contains frequent, intense profanity.
  The 18+ gate is in the app, but the store rating must still be 17+.
  Do not try to rate it lower.
- **Guideline 1.2 / 5.2.3 (User-generated content):** Duke's replies
  come from the built-in engine or the user's own OpenAI key. There is
  no user-to-user content and no sharing, which keeps this simple.
- **Guideline 4.2.6:** the app must be submitted under the user's own
  Apple Developer account — never a freelancer's or agency's account.

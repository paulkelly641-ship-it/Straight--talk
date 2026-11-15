# Straight Talk AI — Duke

A blunt, no-filter AI advice app for adults (18+), built as a plain
static web app and wrapped with **Capacitor** for iOS. It is published
to the App Store with **Codemagic** (cloud Mac builds) from GitHub —
no Mac of your own needed. This project is modeled on the working
My Baby Smarter project (`paulkelly641-ship-it/baby-talk-app`) and the
Potty Mouth project: same Capacitor + Codemagic pattern, same
`appstore` environment group in Codemagic.

## What the app is

Duke, a fictional character who tells it straight — no sugarcoating,
no bullshit, all the cuss words. Ask him anything:

- **Love** — is she right, should you text your ex, is it over?
- **Money** — why you're broke, debt, spending, saving.
- **Work** — your boss, quitting, promotions, getting fired.
- **Life** — motivation, bad habits, feeling stuck.
- **Roast Me** — Duke roasts you, then gives you one real piece of
  advice.

18+ age gate up front (self-confirmation, stored on-device). Saved
chat history (newest first, per-chat delete, clear-all). A built-in
Publish Kit (App Store listing, subscriptions & key, privacy policy,
upload checklist) with copy buttons.

## Duke's two brains

1. **Built-in engine (default):** a local, on-device template/keyword
   reply engine. Blunt, profane, straight talk, topic-keyed answers and
   roasts. Works fully offline and sends nothing anywhere. It is **not**
   a live AI model — it is clearly labelled "Built-in engine" in the
   app. (The hosted web version's built-in engine was server-side
   inference and cannot run in a static app, so this project ships a
   local engine instead. Do not present it as the same thing.)
2. **Real AI (bring-your-own-key):** the user pastes their own OpenAI
   API key in Settings. The app then calls
   `https://api.openai.com/v1/chat/completions` directly (model
   `gpt-4o-mini`, temperature 0.9) from the device. The key is stored
   only in localStorage on the device and is sent **only** to
   api.openai.com — never baked into the app, never sent to any other
   server. The badge always shows which brain is active
   ("Real AI — OpenAI gpt-4o-mini" vs "Built-in engine").

## Free messages & subscription (In-App Purchase)

The first **5 messages are free** (counted across all chats). Then the
paywall appears: unlimited straight talk after subscribing.

| Product  | Product ID                  | Price  | Type                        |
|----------|-----------------------------|--------|-----------------------------|
| Monthly  | `com.paulkelly.straighttalkai.monthly`  | $9.99  | Auto-renewable subscription |
| Yearly   | `com.paulkelly.straighttalkai.yearly`   | $59.99 | Auto-renewable subscription |
| Lifetime | `com.paulkelly.straighttalkai.lifetime` | $99.99 | Non-consumable              |

Bundle ID: `com.paulkelly.straighttalkai`. Monthly & Yearly belong to
subscription group **"Straight Talk Premium"**. Lifetime is a
**non-consumable**, not a subscription.

Purchases go through **Apple In-App Purchase via the RevenueCat Capacitor
plugin** (`@revenuecat/purchases-capacitor` 14.1.0, verified 2026-11-13
for the My Baby Smarter project). The app has **no card form and no web
checkout** — on plain web the paywall shows a preview note instead.
This is required by Apple Guideline 3.1.1.

How the plugin is loaded (no bundler): `www/index.html` loads
`@capacitor/core` 8.6.2 from a pinned CDN build via an **import map**,
then reaches the native RevenueCat plugin with
`Capacitor.registerPlugin('Purchases')`. The plugin's own JS module is
**deliberately not imported**: its ESM files use extensionless relative
imports (`./definitions`) that browsers cannot resolve without a
bundler — this was verified by loading the module in a headless browser
and watching the extensionless fetches fail. `registerPlugin` calls the
exact same native methods the plugin exposes (verified in the plugin's
`PurchasesPlugin.swift`: `configure`, `getOfferings`,
`purchasePackage`, `restorePurchases`, `getCustomerInfo`), with the
same option shapes (`{ apiKey }`, `{ aPackage }`). The native plugin
is installed by `npx cap sync ios` from `package.json`, so on a real
iOS build the bridge resolves `Purchases` to the native code.

One trade-off to know: the Capacitor core module loads from the CDN at
runtime, so the first launch needs internet for purchases to
initialize. The chat and the built-in engine work fully offline
regardless, and if the CDN is unreachable the app degrades gracefully
to preview mode instead of breaking.

## Project layout

```
straight-talk-github/
├── www/index.html              # the whole app (inline CSS + JS, no build step)
├── capacitor.config.json       # appId com.paulkelly.straighttalkai, webDir www
├── package.json                # Capacitor 8 + RevenueCat 14.1.0
├── codemagic.yaml              # ios-workflow: cloud Mac build -> App Store Connect
├── BUILD_STEPS_IOS.md          # full step-by-step for Apple upload
├── store/
│   ├── APP_STORE_LISTING.md    # copy-paste App Store listing
│   ├── privacy-policy.html     # privacy policy page (host on GitHub Pages)
│   └── app-icon-1024.png       # 1024x1024 App Store icon
└── .gitignore                  # node_modules/, ios/, build/
```

The `ios/` folder is **not** in the repo — Codemagic regenerates it on
every build (`rm -rf ios; npx cap add ios; npx cap sync ios`).

## The Codemagic + GitHub flow

1. Create a new GitHub repo (e.g. `straight-talk-app`) and push this project.
2. In Codemagic, add the repo and run the **ios-workflow**. It builds the
   IPA on a cloud Mac mini and uploads it to App Store Connect.
3. In App Store Connect, pick the build and Submit for Review.
   (Full steps: `BUILD_STEPS_IOS.md`.)

The `codemagic.yaml` uses the **`appstore` environment group** — the
same one already configured in Codemagic for Potty Mouth and My Baby
Smarter. It must contain: `APP_STORE_CONNECT_PRIVATE_KEY`,
`APP_STORE_CONNECT_KEY_IDENTIFIER`, `APP_STORE_CONNECT_ISSUER_ID`,
and `CERTIFICATE_PRIVATE_KEY`. Nothing new to configure in Codemagic
if Potty Mouth already builds there.

## One-time setup the user must do himself

These cannot be done from this repo — they need the user's own accounts:

1. **App Store Connect — create the app record**: Straight Talk AI,
   bundle ID `com.paulkelly.straighttalkai` (see `BUILD_STEPS_IOS.md`).
2. **App Store Connect — create the 3 IAP products**: Monthly $9.99 and
   Yearly $59.99 as auto-renewable subscriptions in group
   "Straight Talk Premium"; Lifetime $99.99 as a non-consumable.
   Product IDs are in the table above.
3. **RevenueCat — free account + public key**: sign up at revenuecat.com,
   add the Straight Talk AI app, create an Entitlement named **`premium`**,
   link all three products to it, create an Offering with the three
   packages, then paste the **public Apple API key** (starts with
   `appl_`) into `www/index.html` where it says
   `TODO_REPLACE_WITH_YOUR_REVENUECAT_PUBLIC_APPLE_API_KEY`. The public
   key is safe to ship in the app — never paste a secret key.
4. **App Store Connect — bank & tax info** under Agreements, Tax, and
   Banking (needed before Apple can pay).
5. **App Store Connect API key** — already in Codemagic's `appstore`
   group from Potty Mouth; reuse it, do not create a second one unless
   it expired.

## Honest limits

- The IPA is built by **Codemagic on a cloud Mac**, not on this machine.
  Nothing here is uploaded, submitted, or approved anywhere — Codemagic
  plus the user's own App Store Connect actions do that.
- The native purchase flow can only be fully tested on a **real device
  or TestFlight** with a sandbox Apple ID. The web preview never charges
  a card and never pretends to.
- The built-in engine is a **local template engine, not a live AI**.
  Real AI answers need the user's own OpenAI key (and the user pays
  OpenAI's usage rates).
- **Age rating 17+** (Frequent/Intense Profanity). Apple reviews
  profanity apps strictly, and the same wrapped-web pattern that got
  Potty Mouth rejected under Guideline 4.3(a) (Design — Spam) applies
  here too. This app's concept (interactive advice chat) is more
  distinct than a template song generator, but review can still ask
  for changes — see review tips in `BUILD_STEPS_IOS.md`.
- Every year Apple raises the required Xcode SDK, so expect a rebuild /
  maintenance pass roughly annually.

# AWT Zombie Walk — Mission: On The Move

Landing + registration page for Angels Wings Transport's 4-week community walking
challenge, built on the A Step Ahead™ (Outbreak Challenge) platform.

**Live URL (GitHub Pages):** https://red4golf.github.io/zombie_walk/

This is a single self-contained `index.html` — no build step, no dependencies, no
framework. Just commit and GitHub Pages serves it.

---

## Enabling GitHub Pages

1. Push this repo to GitHub (via GitHub Desktop).
2. On github.com, go to the repo → **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Branch: `main`, folder: `/ (root)`. Save.
5. Wait ~1 minute. Your page goes live at `https://red4golf.github.io/zombie_walk/`.

### (Optional) Custom subdomain — walk.awtrescue.org
If you later want `walk.awtrescue.org` instead of the github.io URL:
1. Add a file named `CNAME` (no extension) to this repo containing one line:
   `walk.awtrescue.org`
2. At your DNS provider, add a CNAME record: `walk` → `red4golf.github.io`
3. In Settings → Pages, enter the custom domain and enable "Enforce HTTPS".
No CNAME file is included yet, since the default github.io URL needs none.

---

## ⚠️ Before it goes live — three things to wire up

The page is fully built and tested, but three placeholders must be filled in
before registration actually works end to end. All three live in the `<script>`
block near the bottom of `index.html`.

### 1. Make.com webhook URL (captures the registration)
Find this line and replace the URL:
```js
const MAKE_WEBHOOK_URL = 'https://hook.us1.make.com/REPLACE_WITH_YOUR_WEBHOOK';
```
Get the real URL by opening the **"AWT Zombie Walk — Registration Intake"**
scenario in Make (scenario 4750195), clicking the webhook trigger, and copying
its address.

### 2. Stripe Payment Link (collects the $6 fee)
Find this line and replace the URL:
```js
const STRIPE_PAYMENT_LINK = 'https://buy.stripe.com/REPLACE_WITH_YOUR_LINK';
```
Create it in your Stripe dashboard:
- **Products → Payment Links → New**
- One-time product, **$6.00**, name it "Mission: On The Move Registration"
- Under "After payment," optionally redirect back to this page
- Create, then copy the `https://buy.stripe.com/...` URL

> **Pricing note:** $6 covers the ~$5 A Step Ahead ticket plus Stripe's
> 2.9% + 30¢ fee (~$0.47), leaving ~$0.53/walker toward comped captain seats.
> True breakeven is $5.49 if you'd rather not fund comps from registration.

### 3. Newsletter subscribe endpoint (only fires if the walker opts in)
This one lives in the **Make** Payment Fulfillment scenario (4750196), not in
this file — the HTTP module that adds opted-in walkers to your newsletter list.
Replace `REPLACE_WITH_NEWSLETTER_SUBSCRIBE_WEBHOOK_OR_MAILCHIMP_ENDPOINT` there.

---

## Other placeholders to update

- **Challenge start date:** the page says "soon" / `[START DATE]` in a couple
  spots — search and replace once you set the real date.
- **App Store / Google Play badges:** currently visual placeholders. Link them to
  the real A Step Ahead™ app listings once your challenge is created.
- **Logo:** the brand mark in the header is the ✈️ emoji. Swap for AWT's real
  logo image if you want pixel-perfect branding.

---

## How the registration flow works

```
Walker fills form  ──►  POST to Make webhook  ──►  row written to Google Sheet
        │                                                  (status: pending_payment
        │                                                   or comp_pending)
        ▼
  Paid walker?  ──►  redirected to Stripe Payment Link ($6)
        │                       │
        │                       ▼
        │              Stripe fires webhook ──► Make matches the sheet row by
        │                                        email, assigns invite code,
        │                                        emails it, adds to newsletter
        │                                        (only if they opted in)
        ▼
  Team organizer (captain)?  ──►  skips payment, lands on "we'll be in touch"
                                   panel; AWT issues a comped code from the
                                   wallet via Make. Comp status is silent —
                                   the page never tells them they're free.
```

### Captain comp logic (already built into this page)
- Creating a **new team** = that person is the team organizer (captain) and is
  silently flagged `isCaptain: true` / `compEligible: true`.
- Joining an **existing team** = regular member (pays).
- **One captain per team** is enforced: trying to "create" a team that already
  has a captain quietly converts to joining it as a member.
- Nothing in the UI mentions free seats. Cap the comps at your 10–20 budget in
  the Make backend, not here.

### Newsletter consent
- The newsletter checkbox is **optional** (checked by default, can be unchecked).
- The email is **always** retained — it's the registration record and how the
  invite code is delivered (transactional, allowed regardless of consent).
- Only `newsletterConsent: true` walkers get added to the marketing newsletter.

---

## Still on the to-do list (tracked separately)

- Build the Google Sheet structure: **Registrations** tab + an **invite-code
  pool** tab.
- Fix the code-assignment step in the Payment Fulfillment scenario to read real
  codes from that pool (currently a placeholder formula).
- Wire the spreadsheet ID, webhook URLs, Stripe link, and account connections
  into both Make scenarios.

---

*Angels Wings Transport is a 501(c)(3) nonprofit. Mission: On The Move uses the
A Step Ahead™ platform; AWT independently organizes this challenge in support of
its mission.*

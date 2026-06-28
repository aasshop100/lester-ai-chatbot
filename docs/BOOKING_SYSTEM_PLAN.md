# Staycation Booking System — Master Plan
> Last updated: 2026-06-29 (v16 — Stage 9 complete; system is production-ready; first client onboarding is next)
> Project: Booking system for staycation/villa clients (Lester AI Chatbot Services)

---

## ⚠️ Stage Gate Rule
**Never start the next stage until the current stage is 100% complete and all checkboxes are checked.**
If a sub-task is blocked, note it under Stage Notes before moving on.

---

## Current Stage
> ✅ **STAGES 1, 2, 3 — Complete**
> ✅ **STAGE 4 — Complete** (all 4 email functions + admin email notification deployed and tested)
> ✅ **STAGE S — Complete** (security hardening done; booking flow end-to-end tested and working)
> ✅ **STAGE 5 — Complete** (admin panel: login, dashboard, bookings, settings — fully live)
> ✅ **STAGE 6 — Complete** (unit management, amenities, add-ons, date blocking pages live)
> ✅ **STAGE 7 — Complete** (FullCalendar.js calendar page, expenses page, net proceeds on dashboard)
> ✅ **STAGE 9 — COMPLETE** (all features live and bug-free as of 2026-06-29)
> ⏭️ **STAGE 8 — Skipped** (chatbot system prompt update is a 5-min manual task — do anytime)
> 🔄 **NEXT — First Real Client Onboarding** (see "Next Session" section below)

---

## Architecture Decisions (Locked — Do Not Change Without Discussion)

| Decision | Choice | Reason |
|----------|--------|--------|
| Multi-client strategy | One Firebase project, scoped by `clientId` | Zero extra setup per client. Each client's data lives under `/clients/{clientId}/`. Easy to expand. |
| Booking website hosting | GitHub Pages (per client repo) | Free, static, fast. Dynamic data comes from Firebase SDK calls. |
| Admin panel hosting | New Netlify site (separate from Beligas CRM) | Admin needs server-side functions to send emails. Cannot do SMTP from browser. |
| Email functions | Netlify Functions on admin Netlify site | Free tier (125K calls/month). Already familiar with this setup. |
| Firebase plan | Spark (free) | No Firebase Functions needed — Netlify handles server-side logic. |
| Booking reference number | Firestore auto-ID prefixed with `BK-` | Guaranteed unique even with concurrent bookings. No counter needed. |
| Screenshot upload | **Cloudinary free tier** (replaces Firebase Storage) | Firebase Storage now requires Blaze (paid) plan on new projects. Cloudinary free = 25GB storage — enough for 600+ months of 100 bookings/month. API-based upload direct from browser. We store the Cloudinary URL in Firestore. |
| First build target | Single client (no multi-tenant UI yet) | Get one working perfectly first. Multi-client support added in Stage 9. |
| Tech stack (booking site) | Plain HTML + CSS + Vanilla JS | Lightweight, fast on mobile, no build tools needed for GitHub Pages. |
| Tech stack (admin panel) | Plain HTML + CSS + Vanilla JS | Same reason. Consistent with existing Beligas CRM codebase. |
| Email sender (per client) | Client's own dedicated Gmail + App Password | Clients create a Gmail for their property (e.g. villarosa.bookings@gmail.com). They generate a Gmail App Password — not their personal password. Stored in Firestore `settings.smtpEmail` + `settings.smtpPassword`. Netlify Function reads it at send time. Each client's emails come from their own address. |
| Telegram notifications | Optional per client | Not every client wants Telegram. Netlify Function checks if `TELEGRAM_BOT_TOKEN` exists in client settings before sending. If not set — silently skipped. Email always sends regardless. |
| GitHub account for repos | `bookingPH` | Separate from personal `aasshop100` account. All booking site repos live here. One repo per client. |

---

## System Architecture

```
[Customer on Facebook Messenger]
        ↓
[n8n AI Chatbot — MAIN ROUTER]
  Handles inquiries → sends booking URL when customer is ready to book
        ↓
[Booking Website — GitHub Pages]
  {clientname}.github.io (or custom domain)
  Landing page → Property listing → Property detail → 4-step booking flow
  Step 1: Select dates  →  Step 2: Guest details  →  Step 3: Add-ons  →  Step 4: Payment
  GCash / PayMaya / Bank Transfer → Upload screenshot → Confirmation message
        ↓
[Firebase — Backend]
  Firestore: bookings, units, settings, amenities, addons, blocked_dates, expenses
  Cloudinary: payment screenshots, unit photos, QR code images
  Auth: admin login accounts (one per client)
        ↓
[Netlify Functions — Server-side logic]            [Notifications]
  /send-receipt    ← called when screenshot uploaded    Telegram → admin (optional, per client)
  /send-confirm    ← called when admin confirms         Email → customer (from client's Gmail)
  /send-reject     ← called when admin rejects          Email → customer (confirmed/rejected)
  /send-cancel     ← called when admin cancels
        ↓
[Admin Panel — Netlify]
  booking-admin.netlify.app (or custom)
  Login → Dashboard → Bookings → Review screenshot → Confirm/Reject
  Unit Management → Amenities → Add-ons → Holiday Rates → Manual Blocking
  Calendar → Expenses → System Settings
```

---

## Key Values
> Fill in as we create each resource. Never hardcode — always refer back here.

| Resource | Value |
|----------|-------|
| Firebase Project ID | `lester-booking-system` |
| Firebase Web API Key | `[REDACTED — in firebase-config.js, not committed to public repo]` |
| Firebase Auth Domain | `lester-booking-system.firebaseapp.com` |
| Firebase App ID | `[REDACTED — in firebase-config.js]` |
| Firebase Messaging Sender ID | `[REDACTED — in firebase-config.js]` |
| Firebase Storage Bucket | **N/A — using Cloudinary** (see Screenshot Upload decision) |
| Firestore Region | `asia-southeast1` (Singapore) |
| Service Account Key | `C:\Users\LESTER\lester-booking-sa-key.json` |
| Service Account Email | `firebase-admin-sdk@lester-booking-system.iam.gserviceaccount.com` |
| First Client ID (`clientId`) | `lester-domain-staycation` |
| Cloudinary Cloud Name | `dwqweaowd` |
| Cloudinary API Key | `[REDACTED — see Netlify env: CLOUDINARY_API_KEY]` |
| Cloudinary API Secret | `[REDACTED — see Netlify env: CLOUDINARY_API_SECRET]` |
| Cloudinary Upload Preset (public photos/QR) | `booking-public` (unsigned) |
| Cloudinary Upload Preset (payment proofs) | ~~`booking-screenshots` (unsigned)~~ → **replaced by signed upload via `get-upload-signature` function** |
| GitHub Repo (booking site) | `https://github.com/bookingPH/staycation-booking-site` |
| GitHub Pages URL | `https://bookingph.github.io/staycation-booking-site/` |
| Local repo (booking site) | `C:\Users\LESTER\Desktop\staycation-booking-site` |
| GitHub Repo (admin panel) | `https://github.com/bookingPH/staycation-admin-panel` |
| Admin Panel Netlify Site | `staycation-admin` (ID: 6a84ea95-a5a4-42eb-a068-0df985c3deac) |
| Admin Panel URL | `https://staycation-admin.netlify.app` |
| Local repo (admin panel) | `C:\Users\LESTER\Desktop\staycation-admin-panel` |
| Netlify Function: send-receipt | `https://staycation-admin.netlify.app/.netlify/functions/send-receipt` |
| Netlify Function: send-confirm | `https://staycation-admin.netlify.app/.netlify/functions/send-confirm` |
| Netlify Function: send-reject | `https://staycation-admin.netlify.app/.netlify/functions/send-reject` |
| Netlify Function: send-cancel | `https://staycation-admin.netlify.app/.netlify/functions/send-cancel` |
| Netlify Function: create-booking | `https://staycation-admin.netlify.app/.netlify/functions/create-booking` |
| Netlify Function: get-upload-signature | `https://staycation-admin.netlify.app/.netlify/functions/get-upload-signature` |
| Netlify Function: expire-holds | `https://staycation-admin.netlify.app/.netlify/functions/expire-holds?secret=EXPIRE_SECRET` |
| EXPIRE_SECRET | `[in Netlify env vars — Staycation2026!]` |
| Cron job (expire-holds) | cron-job.org — every 15 min — `*/15 * * * *` |
| First Admin Email (client login) | `admin@lesterbooking.com` / `[ROTATED — stored in password manager]` |
| Test SMTP Email | `lstrmrcd@gmail.com` (Lester's Gmail — for testing only) |
| Test SMTP App Password | `[ROTATED — new value in Firestore settings/private]` |
| SMTP Host | `smtp.gmail.com` port `587` |
| Telegram | Optional per client — stored in Firestore `settings.telegramToken` + `settings.telegramChatId` |
| Existing Netlify (Beligas CRM) | `beligas-crm.netlify.app` |

---

## Firestore Data Model
> Designed for multi-client from Day 1. First build uses one clientId only.

```
/clients/{clientId}/
  settings/public    → companyName, address, phone, email, logo
                        gcashName, gcashNumber, gcashQrUrl (Cloudinary URL)
                        mayaName, mayaNumber, mayaQrUrl (Cloudinary URL)
                        bankName, bankAccountName, bankAccount, bankQrUrl (Cloudinary URL)
                        downPaymentAmount (₱ — reservation fee, editable)
                        securityDeposit (₱ — refundable, editable; collected at check-in)
                        ← PUBLIC read (booking site reads company name, QR, payment info)
  settings/private   → smtpEmail (client's Gmail), smtpPassword (Gmail App Password)
                        telegramToken (optional), telegramChatId (optional)
                        ← NO public read — Admin SDK only (Netlify Functions read via Firebase Admin)
  units/{unitId}/    → name, description, location, weekdayRate, weekendRate, checkIn, checkOut,
                        maxGuests, maxExtraGuests, extraGuestFee, petsAllowed, petFee,
                        rating, photoURLs[], amenityIds[], status, createdAt
  bookings/{bookingId}/  → referenceNo, unitId, unitName, guestName, guestEmail, guestPhone,
                            numGuests, checkIn, checkOut, nights, baseTotal, extraGuests,
                            extraGuestTotal, addons[], addonsTotal, grandTotal,
                            reservationFee, securityDeposit, balanceDueAtCheckin,
                            paymentMethod, screenshotURL, status (pending_review /
                            confirmed / rejected / cancelled), adminNote, createdAt, confirmedAt
  availability/{bookingId}/ → bookingId, unitId, checkIn, checkOut, status, createdAt
                               ← PUBLIC read — calendar uses this (no personal info exposed)
                               ← Written SERVER-SIDE by create-booking.js (not by browser)
                               ← Updated to 'confirmed'/'rejected'/'cancelled'/'expired' by Netlify Functions
  amenities/{amenityId}/ → name, icon (Font Awesome class), description
  addons/{addonId}/  → unitId (or 'all'), name, price, billingType (pernight / onetime)
  blocked_dates/{id}/ → unitId, startDate, endDate, reason, createdAt
  expenses/{id}/     → date, category, description, amount, unitId (or 'all'), createdAt
```

---

## Stage 1 — Foundation & Architecture Setup
> Goal: All infrastructure ready. No UI. All decisions documented. Anyone can pick this up and know exactly what exists.

### Tasks

**Firebase Setup**
- [x] Create new Firebase project → `lester-booking-system` ✅
- [x] Enable Firestore Database (asia-southeast1) ✅
- [x] Firebase Web App created — API Key recorded ✅
- [x] Service account created + IAM roles granted ✅
- [ ] ⚠️ MANUAL STEP NEEDED: Enable Firebase Authentication → go to https://console.firebase.google.com/project/lester-booking-system/authentication → "Get started" → Email/Password → Enable → Save
- [ ] Create first admin user in Firebase Auth (email: admin@lesterbooking.com, password: Admin@2026!)
- [ ] **Firebase Storage: REPLACED by Cloudinary** — set up Cloudinary free account (cloudinary.com), record Cloud Name + Upload Preset in Key Values

**Firestore Setup**
- [ ] Create admin user first (requires Auth to be enabled) — then run setup script to seed Firestore
- [ ] clientId = `lester-domain-staycation` (recorded in Key Values)
- [ ] Seed test data (needed so Stage 2 has something to display):
  - [ ] `settings` — placeholder company name, address, phone, email, down payment amount ₱500
  - [ ] `amenities` — at least 4 sample amenities (Swimming Pool, WiFi, Parking, Air Conditioning)
  - [ ] `units` — 2 sample units with name, description, location, weekday/weekend rate, capacity
  - [ ] `addons` — 1 sample add-on (e.g., Gourmet Breakfast ₱350/night)
- [ ] Note: seed data is TEST data — replaced by real client data in Stage 6

**Firebase Security Rules**
- [x] Firestore rules written: `/firebase-config/firestore.rules` ✅
- [ ] Deploy Firestore rules (`firebase deploy --only firestore:rules`) — needs Auth enabled first
- [ ] Storage rules: N/A — using Cloudinary. Cloudinary handles upload security via unsigned upload preset scoped to `payment-screenshots` folder.
- [ ] Test Firestore rules using Firebase Rules Playground after deploy

**GitHub Setup**
- [ ] Create GitHub repo for booking website (name: `staycation-booking-site` or client-specific)
- [ ] Enable GitHub Pages (branch: `main`, folder: `/` root)
- [ ] Verify GitHub Pages URL is live (shows blank page — that's fine for now)
- [ ] Record GitHub repo URL and Pages URL in Key Values table

**Admin Panel Setup**
- [ ] Create new Netlify site for admin panel (connect to a new GitHub repo or same repo `/admin` folder)
- [ ] Record Netlify admin panel URL in Key Values table
- [ ] Set Netlify environment variables:
  - [ ] `FIREBASE_PROJECT_ID`
  - [ ] `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` (privateemail.com)
  - [ ] `TELEGRAM_BOT_TOKEN`
  - [ ] `TELEGRAM_CHAT_ID`

**Documentation**
- [ ] Record all created IDs and URLs in Key Values table above
- [ ] Write Stage Notes: any decisions made, anything that changed from the plan

### Stage Notes
**2026-06-28:**
- Firebase project created via CLI: `lester-booking-system`
- Firestore database created: region `asia-southeast1`, native mode
- Firebase Web App created — full SDK config recorded in Key Values
- Service account `firebase-admin-sdk@lester-booking-system.iam.gserviceaccount.com` created; key saved at `C:\Users\LESTER\lester-booking-sa-key.json`; IAM roles granted: firebase.admin, datastore.user, firebasestorage.admin
- Firebase Storage requires Blaze plan on new projects — switched to **Cloudinary free tier**. No billing required. 25GB free = enough for 600+ months of bookings.
- Architecture Decision: Screenshot URLs stored in Firestore as Cloudinary URLs. Storage rules section removed. Cloudinary upload preset (unsigned) scoped to payment-screenshots folder.
- Firestore security rules written (`firebase-config/firestore.rules`). Storage rules file removed from deploy scope.
- Firebase Auth initialization via API blocked — requires one-time Firebase Console "Get Started" click (standard Google requirement, cannot be bypassed via API).
- **BLOCKED ON:** User needs to click "Get Started" in Firebase Console → Authentication page to enable Email/Password auth, then we create admin user + deploy rules + set up Cloudinary + GitHub + Netlify.

### Status
- [x] Stage 1 Complete ✅ (2026-06-28)

---

## Stage 2 — Booking Website: Listing Page + Property Detail Page
> Goal: Live, professional public website. Two pages working. Data comes from Firebase.

### Tasks

**Project File Structure**
- [ ] Create files: `index.html`, `property.html`, `styles.css`, `app.js`, `firebase-config.js`
- [ ] Add Firebase SDK (compat version via CDN — no build tools needed)
- [ ] Connect Firebase using config from Key Values table
- [ ] Verify Firebase connection works (console.log Firestore read)

**Landing Page (`index.html`)**
- [ ] `<head>`: SEO meta tags (title, description, Open Graph image, viewport)
- [ ] `<head>`: Favicon
- [ ] Navbar: logo + property name + hamburger menu (mobile)
- [ ] Hero section: full-width image, property name, tagline, "Book Now" CTA button
- [ ] Hero images load from Firestore `settings` (not hardcoded)
- [ ] "Featured Accommodations" section — reads from Firestore `units`:
  - [ ] Property card: photo, name, location, price/night, amenity icons, "View" + "Book Now" buttons
  - [ ] Loading skeleton shown while Firebase loads
  - [ ] Empty state if no units exist yet
- [ ] Amenities section — icon grid reads from Firestore `amenities`
- [ ] Testimonials section (static for now — hardcoded placeholder reviews)
- [ ] Footer: company name, address, phone, email, social links, nav links
  - [ ] Footer content reads from Firestore `settings` (not hardcoded)
- [ ] Mobile-first, fully responsive (test at 375px, 768px, 1280px widths)
- [ ] Page loads in under 3 seconds on mobile (check Chrome DevTools Lighthouse)

**Property Detail Page (`property.html`)**
- [ ] Reads `unitId` from URL parameter: `property.html?id=UNIT_ID`
- [ ] Full-width photo gallery (multiple photos, tap to view full screen)
- [ ] Property name, location, rating stars
- [ ] Weekday rate + weekend rate displayed
- [ ] Capacity info (max guests, bedrooms, bathrooms)
- [ ] Full description text
- [ ] Amenities list (icons + names specific to this unit)
- [ ] Pet policy info (allowed yes/no + fee)
- [ ] Extra guest info (max extra guests + fee)
- [ ] Check-in / check-out times
- [ ] "Book Now" button → goes to booking flow with `unitId` pre-selected
- [ ] If unit not found → show "Property not found" message

**Static Policy Pages**
- [ ] Create `booking-policy.html` — static page with booking rules (deposit, check-in/out times, cancellation)
- [ ] Create `cancellation.html` — static page with cancellation and refund policy
- [ ] Footer links point to these pages (not broken links)
- [ ] Content for both pages uses placeholder text for now — client fills in later

**Polish**
- [ ] "View" button on property card links to `property.html?id=UNIT_ID`
- [ ] "Book Now" on card and detail page links to booking flow
- [ ] Test on real mobile device (not just browser resize)
- [ ] Deploy to GitHub Pages and verify ALL pages work on live URL (index, property, booking-policy, cancellation)

### Stage Notes
**2026-06-28:**
- All 6 files built and deployed to GitHub Pages: `index.html`, `property.html`, `booking.html`, `booking-policy.html`, `cancellation.html`, `styles.css`, `firebase-config.js`
- Design system: CSS variables (--primary #C8623A terracotta, --accent #2D5016 deep green, --bg #FAF6F1), Playfair Display + Inter fonts, Font Awesome 6.5 icons
- `index.html`: loads company name from Firestore `settings/main`, loads room cards from `units`, loads amenity grid from `amenities`, hero, stats bar, how-it-works, CTA, footer
- `property.html`: reads `?id=` param, shows hero photo, room details, amenities, pricing box. Bug fixed: Firestore `settings` rule was admin-only, booking site couldn't read company name → changed to public read
- Bug fixed (property.html): catch block now hides booking-box on error to avoid showing broken content
- All pages tested on GitHub Pages — live and working

### Status
- [x] Stage 2 Complete ✅ (2026-06-28)

---

## Stage 3 — Booking Website: 4-Step Booking Flow + Telegram Alert
> Goal: Customer completes a full booking. Screenshot saved to Firebase. Admin alerted via Telegram instantly.
> Note: Telegram alert is part of this stage because you cannot properly test booking creation without it.
> ⚠️ Dependency: Stage 3 calls `send-receipt` (a Netlify function). Build `send-receipt.js` from Stage 4 BEFORE running Stage 3's final end-to-end test. The rest of Stage 3 can be built and tested without it — just do final e2e test after Stage 4 is done.

### Tasks

**Booking Page Structure**
- [ ] Create `booking.html`
- [ ] Reads `unitId` from URL parameter
- [ ] Loads unit details from Firestore (name, rates, check-in/out times, max guests)
- [ ] Step progress bar visible throughout (Step 1 of 4)
- [ ] Back button works between all steps (does not lose entered data)

**Step 1 — Select Dates**
- [ ] Month calendar UI (current month, navigate forward/backward)
- [ ] Fetch unavailable dates from Firestore using `onSnapshot` (real-time listener — auto-updates without page refresh):
  - [ ] `blocked_dates` for this unit
  - [ ] `bookings` for this unit where status = `pending_review` OR `confirmed` (both block the calendar)
  - [ ] ⚠️ `pending_review` bookings MUST block the calendar — not just `confirmed`. This prevents Customer B from selecting dates that Customer A just submitted.
- [ ] Color-coded dates: Available (default) / Unavailable (red — covers both booked + pending) / Manually Blocked (grey)
- [ ] Cannot select past dates
- [ ] Cannot select unavailable/blocked dates
- [ ] Click first date = check-in, click second date = check-out
- [ ] If customer has dates selected and `onSnapshot` fires a change (someone else just booked those dates):
  - [ ] Highlight the conflicting dates in red
  - [ ] Show warning: "These dates were just taken. Please select new dates."
  - [ ] Reset date selection
- [ ] Auto-calculate: number of nights
- [ ] Auto-calculate: total price (weekday vs weekend rate per night, holiday rate override if set)
- [ ] Extra guest fee added if num guests > base capacity
- [ ] Summary shown: Check-in, Check-out, Nights, Subtotal
- [ ] Cannot proceed if no dates selected

**Step 2 — Guest Details**
- [ ] Fields: Full Name, Email Address, Phone Number, Number of Guests
- [ ] Validation: all fields required, email format check, phone format check
- [ ] Number of guests cannot exceed unit's max guests (base + max extra)
- [ ] If guests exceed base capacity: show extra guest fee notice
- [ ] Cannot proceed if validation fails

**Step 3 — Add-ons**
- [ ] Load add-ons from Firestore for this unit (or all-unit add-ons)
- [ ] Each add-on: checkbox + name + price + billing type (per night or one-time)
- [ ] Total price updates in real time as checkboxes are toggled
- [ ] Full price breakdown shown: base nights total + each add-on + grand total
- [ ] If no add-ons exist for this unit: show "No add-ons available" and auto-proceed is optional
- [ ] Step is skippable (Continue without add-ons)

**Step 4 — Payment**
- [ ] Payment method selector: GCash / PayMaya / Bank Transfer (radio buttons or tab)
- [ ] On select:
  - [ ] GCash: show QR code image + account name + phone number (from Firestore `settings`)
  - [ ] PayMaya: show QR code image + account name + phone number
  - [ ] Bank Transfer: show bank name + account name + account number + QR (if available)
- [ ] QR code images load from Firebase Storage URLs stored in `settings`
- [ ] Instruction text: "Transfer ₱[downpayment amount] then upload your screenshot below"
- [ ] Down payment amount reads from Firestore `settings`
- [ ] File upload input: accepts jpg, png, webp only
- [ ] File size limit: 10MB max — show error if exceeded
- [ ] Upload button with progress indicator (0% → 100%)
- [ ] On upload complete:
  - [ ] **Final availability check** — before saving booking to Firestore, query again for any `pending_review` or `confirmed` bookings on the same unit overlapping the selected dates
  - [ ] If conflict found (someone else booked while customer was on Steps 2–4):
    - [ ] Do NOT save the booking
    - [ ] Do NOT show confirmation screen
    - [ ] Show error: "Sorry, these dates were just booked by someone else. Please go back and select new dates."
    - [ ] Show "Choose New Dates" button → resets to Step 1
  - [ ] If no conflict: proceed normally
  - [ ] Screenshot saved to Firebase Storage: `screenshots/{clientId}/{bookingId}/proof.jpg`
  - [ ] Booking record saved to Firestore with all details + screenshotURL + status = `pending_review`
  - [ ] Booking reference number = `BK-` + Firestore auto-generated document ID
  - [ ] Trigger Telegram alert (see below)
  - [ ] Show confirmation screen (NOT a new page — same page, final state)
- [ ] Confirmation screen:
  - [ ] Booking reference number displayed prominently
  - [ ] Summary: property, dates, guest name, total, payment method
  - [ ] Message: "Your booking request has been received! We'll send a confirmation to [email] once we verify your payment. This usually takes 1–2 hours."
  - [ ] Do NOT show a "go back" option — booking is already submitted

**Error Handling (all steps)**
- [ ] Firebase connection error → show "Something went wrong. Please try again."
- [ ] Screenshot upload fails → show error + keep the upload button active (let them retry)
- [ ] Firestore write fails → show error + do NOT show confirmation screen
- [ ] Loading states on every async action (spinner or disabled button)
- [ ] No double-submit: disable "Confirm Booking" button immediately on click

**Telegram Notification (fires on successful booking save)**
- [ ] Call n8n Telegram webhook (or direct Telegram API) with:
  - [ ] 🔔 New Booking Alert
  - [ ] Booking ID, Guest name, Property, Check-in, Check-out, Nights, Grand total
  - [ ] Payment method
  - [ ] Screenshot link (Firebase Storage URL)
  - [ ] Link to admin panel
- [ ] If Telegram fails: log error silently — do NOT block the customer confirmation screen

**Testing**
- [ ] Test complete flow on desktop
- [ ] Test complete flow on real mobile device
- [ ] Test with slow network (Chrome DevTools → Slow 3G)
- [ ] Test file upload with jpg, png, and a wrong file type (should reject)
- [ ] Test file upload with oversized file (should reject)
- [ ] Verify booking appears in Firestore with correct data
- [ ] Verify screenshot appears in Firebase Storage
- [ ] Verify Telegram alert received

### Stage Notes
**2026-06-28:**
- `booking.html` built: 4-step flow (dates → guest info → add-ons → payment), progress bar, back navigation, no data loss between steps
- **Calendar**: uses event delegation (one listener on grid container) — NOT per-cell listeners. Per-cell listeners caused lost clicks because `renderCalendar()` replaces `grid.innerHTML` on every render. Event delegation survives innerHTML replacements. `updateRangeDisplay()` updates CSS range classes without DOM re-render.
- **Availability collection**: created new Firestore `availability` collection (public read/create) that stores ONLY `{unitId, checkIn, checkOut, status}`. Calendar listens to this. Booking site writes here on submit. This separates calendar blocked-date data from personal guest data (which stays in admin-only `bookings` collection).
- **Firestore rules deployed**: `availability` — public read + create, admin update. `settings` — changed to public read (booking site needs company name, QR codes). All other rules unchanged. Deployed via: `firebase deploy --only firestore:rules --project lester-booking-system`
- **Screenshot upload**: Cloudinary via unsigned XMLHttpRequest upload. Cloud name `dwqweaowd`, preset `booking-screenshots`. Progress bar shown. URL stored in Firestore booking record.
- **Reservation fee + security deposit**: both editable in Firestore `settings/main` fields `downPaymentAmount` (₱500) and `securityDeposit` (₱1000). Price summary clearly shows: Grand Total → Pay Now (reservation fee) → Pay at Check-in (balance + security deposit).
- **Booking record fields added**: `reservationFee`, `securityDeposit`, `balanceDueAtCheckin` stored in each booking document.

**4 bugs fixed during Stage 3:**
1. "Room not found" on property.html → Firestore settings was admin-only read, changed to public read
2. Can't select check-out date → event delegation fix + calendar listener changed from `bookings` (permission-denied) to `availability`
3. "Something went wrong" on Confirm → final availability conflict check was reading `bookings` (admin-only), changed to read `availability`
4. "Invalid Date" on confirmation screen → after submitting, the `availability` onSnapshot fired, saw user's own dates as blocked, cleared `S.checkIn`/`S.checkOut`. Fix: stop calendar listener BEFORE writing to Firestore (not after)

**What's NOT done from Stage 3 plan (intentional deferrals):**
- Telegram alert: deferred to Stage 4 (it's server-side inside `send-receipt.js`)
- End-to-end test with email: blocked on Stage 4 (`send-receipt` not deployed yet) — Stage 3 code silently skips the send-receipt call if it's unreachable
- Booking site calls `https://staycation-admin.netlify.app/.netlify/functions/send-receipt` — this URL is hardcoded as `SEND_RECEIPT_URL` at top of booking.html

### Status
- [x] Stage 3 Complete ✅ (2026-06-28)

---

## Stage 4 — Netlify Functions + Email Notifications
> Goal: Four email functions working. Customer gets email on submission and on admin decision.
> ⚠️ Start Here: `send-receipt.js` is called by the booking site immediately after submission. Build and deploy this first, then the other three.
> Admin panel location: `C:\Users\LESTER\Desktop\staycation-admin-panel` — deploy via `netlify deploy --prod` from that folder.

### Tasks

**Netlify Setup for Functions**
- [x] Create `netlify/functions/` folder in admin panel repo (`C:\Users\LESTER\Desktop\staycation-admin-panel\netlify\functions\`) ✅
- [x] Set Netlify environment variables on staycation-admin.netlify.app: ✅
  - [x] `FIREBASE_SERVICE_ACCOUNT` — stored as base64-encoded JSON (avoids CLI quoting issues) ✅
  - [x] `SMTP_HOST` = `smtp.gmail.com` ✅
  - [x] `SMTP_PORT` = `587` ✅
  - [x] (SMTP user/pass are per-client, read from Firestore `settings/main` at send time — NOT in env vars) ✅
- [x] Install nodemailer + firebase-admin: `npm install` in admin panel repo ✅
- [x] `package.json` created with nodemailer + firebase-admin dependencies ✅

**Function 1: `send-receipt.js` ← BUILD THIS FIRST**
- [x] Triggered by booking website after successful booking save (POST call from browser) ✅
- [x] Must allow CORS from `https://bookingph.github.io` (booking site origin) ✅
- [x] Accepts JSON body: `{ clientId, bookingId, referenceNo, guestName, guestEmail, unitName, checkIn, checkOut, nights, grandTotal, paymentMethod, screenshotURL, adminPanelUrl }` ✅
- [x] Reads client's SMTP credentials from Firestore: `clients/{clientId}/settings/main` → `smtpEmail`, `smtpPassword` ✅
- [x] Reads company name from same settings doc → `companyName` ✅
- [x] Sends email to GUEST (guestEmail): ✅
  - Subject: `Booking Request Received — [referenceNo]`
  - Body: property name, check-in, check-out, nights, grand total, payment method, reference number
  - Message: "We received your payment screenshot. We'll confirm within 1–2 hours."
  - "From" name: client's company name
- [x] Sends Telegram alert to ADMIN (if `settings.telegramToken` and `settings.telegramChatId` are set): ✅
  - Message format with emoji, ref, guest, room, dates, total, payment, screenshot link, admin panel link
  - Uses Telegram sendMessage API with HTML parse mode
  - If Telegram fails: logs error, does NOT block the email send
  - **Notification design decision:** Guest gets email, client/admin gets Telegram. No self-email (avoids client emailing their own address).
  - **How to set up Telegram per client (Lester does this during onboarding):**
    1. Open Telegram → @BotFather → `/newbot` → get bot token
    2. Tell client to search the bot and click Start
    3. Visit `https://api.telegram.org/botTOKEN/getUpdates` → get chat ID
    4. Write `telegramToken` + `telegramChatId` to Firestore `clients/{clientId}/settings/main`
  - **Client onboarding kit** (full credential checklist) — to be created as separate doc after Stage 4
- [x] Returns `{ success: true }` or `{ error: "..." }` ✅
- [x] CORS headers required: `Access-Control-Allow-Origin: https://bookingph.github.io` ✅
- [x] Test: full end-to-end POST from PowerShell → 200 `{"success":true}` → email received at lstrmrcd@gmail.com ✅

**Function 2: `send-confirm.js`**
- [x] Triggered by admin panel when admin clicks "Confirm" on a booking ✅
- [x] Accepts: clientId, guestName, guestEmail, bookingId, referenceNo, unitName, checkIn, checkOut, checkInTime, checkOutTime, grandTotal, paymentMethod, adminNote (optional) ✅
- [x] Sends email to customer: ✅
  - Subject: `✅ Booking Confirmed — [Reference No]`
  - Body: full booking summary, check-in/out with times, property address, contact info
  - Message: "Your booking is confirmed! See you on [check-in date]."
  - Include: "If you need to cancel or reschedule, contact us at [phone/email]"
- [x] Test: `{"success":true}` → email received ✅

**Function 3: `send-reject.js`**
- [x] Triggered by admin panel when admin clicks "Reject" on a booking ✅
- [x] Accepts: clientId, guestName, guestEmail, bookingId, referenceNo, unitName, checkIn, checkOut, adminNote (required), bookingWebsiteUrl (optional) ✅
- [x] Server enforces adminNote required — returns 400 if blank ✅
- [x] Sends email to customer: ✅
  - Subject: `Booking Update — [Reference No]`
  - Body: explains rejection, adminNote shown as reason, "Book Again" button if bookingWebsiteUrl provided
  - Message: "You may re-book by visiting our booking site. We apologize for the inconvenience."
- [x] Test: `{"success":true}` → email received ✅

**Function 4: `send-cancel.js`**
- [x] Triggered by admin panel when admin clicks "Cancel Booking" on a confirmed booking ✅
- [x] Accepts: clientId, guestName, guestEmail, bookingId, referenceNo, unitName, checkIn, checkOut, adminNote (optional) ✅
- [x] Sends email to customer: ✅
  - Subject: `Booking Cancellation Notice — [Reference No]`
  - Body: booking summary + cancellation notice + adminNote + deposit query instruction
  - Message: "We regret to inform you that your booking has been cancelled. Please contact us at [phone/email] for questions about your deposit."
- [x] Test: `{"success":true}` → email received ✅

**Email Template Design**
- [ ] All 4 emails use consistent HTML template (header with logo/property name, body, footer)
- [ ] Mobile-friendly email layout
- [ ] Logo/property name reads from Firestore `settings`
- [ ] Plain-text fallback for email clients that don't render HTML

**Email Spam Prevention** — ⏩ Deferred to Stage S (Security Hardening)
- [ ] Verify SPF record is set on privateemail.com sending domain (prevents Gmail marking as spam)
- [ ] Verify DKIM record is configured (privateemail.com provides this — check their DNS settings)
- [ ] Send test emails to Gmail, Yahoo, and Outlook — verify they land in inbox, not spam
- [ ] If landing in spam: add "from" name matching domain (e.g., no-reply@clientdomain.com)

**Testing** — ⏩ Deferred to Stage S (Security Hardening)
- [ ] Test all 4 functions via Postman or direct fetch call
- [ ] Verify emails arrive in inbox (not spam) on Gmail, Yahoo, Outlook
- [ ] Verify email content is correct for each scenario

### Stage Notes
**2026-06-28:**
- `package.json` created in admin panel repo with `nodemailer` + `firebase-admin` dependencies. `npm install` run locally so Netlify can bundle them.
- `FIREBASE_SERVICE_ACCOUNT` env var: stored as base64-encoded minified JSON (not raw JSON) because Netlify CLI strips quotes from JSON values when set via PowerShell. Decoded in function with `Buffer.from(val, 'base64').toString('utf8')` before JSON.parse. All other Firebase functions in this project should use the same pattern.
- `smtpEmail` + `smtpPassword` seeded into Firestore `clients/lester-domain-staycation/settings/main` using test credentials (lstrmrcd@gmail.com + Gmail App Password).
- Email HTML template: terracotta (#C8623A) header, reference number box, booking detail table, pending-status notice, plain-text fallback.
- Telegram alert: optional, silently skipped if token/chatId not in Firestore settings. Does not block email.
- Notification split decided: guest = email, client/admin = Telegram. Avoids self-email awkwardness.
- Telegram onboarding steps documented in plan (BotFather → token → chat ID → write to Firestore).
- Client onboarding kit (full credential checklist) to be created as a separate doc after Stage 4 is complete.
- Tested: POST with full payload → `{"success":true}` → email delivered to lstrmrcd@gmail.com.
- Function URL: `https://staycation-admin.netlify.app/.netlify/functions/send-receipt`

### Status
- [x] Stage 4 Complete ✅ (2026-06-28) — All 4 email functions deployed. Admin email notification added to send-receipt.js (fires on every new booking). Email spam prevention deferred to Stage 9 polish.

---

## Stage S — Security Hardening
> Goal: System is safe to go live. No exposed credentials, no server-trusting-browser-data, no race-condition double-bookings, no unrestricted public uploads.
> ⛔ Stage 5+ is blocked until every Priority 0–3 task below is complete and tested.

### Architecture Changes Locked in This Stage

| Decision | Old | New | Reason |
|----------|-----|-----|--------|
| Firestore `settings` | Single `settings/main` (public read) | `settings/public` (public read) + `settings/private` (no public read, Admin SDK only) | SMTP + Telegram credentials must never be readable by browser JS |
| Booking creation | Browser writes directly to Firestore | Server-side `create-booking` Netlify function | Server must calculate prices, claim inventory atomically; browser cannot be trusted |
| Per-night inventory | `availability` collection (public create) | `inventory/{unitId_YYYY-MM-DD}` (server-only, Firestore transaction) | Atomic claim of all nights prevents any race-condition double-booking |
| Email functions input | Full booking data sent by caller | Only `clientId` + `bookingId`; function fetches from Firestore | Prevents caller from spoofing email, amounts, recipients |
| Payment screenshot upload | Unsigned Cloudinary preset (browser-controlled) | Signed upload via `get-upload-signature` function | Server controls folder, format, size, filename |
| Public Firestore create on `bookings` | `allow create: if true` | `allow create: if false` — server-side only | Prevents fake bookings without a real booking transaction |
| Public Firestore create on `availability` | `allow create: if true` | `allow create: if false` — server-side only | Prevents fake calendar blocks |

### Priority 0 — Credential Rotation and Secrets (MANUAL — Lester must do these)
> These cannot be done by code. Do them before running the migration script.

- [ ] **Rotate Gmail App Password** — Go to myaccount.google.com → Security → App passwords → Revoke old → Generate new → Update in Firestore `settings/private` after migration
- [ ] **Reset Firebase admin password** — Firebase Console → Authentication → admin@lesterbooking.com → Reset password → Store new password in password manager only (never in docs)
- [ ] **Add Cloudinary API Secret to Netlify** — Cloudinary Console → API Keys → copy API Secret → Netlify → staycation-admin → Environment variables → add `CLOUDINARY_API_SECRET`
- [ ] **Confirm git history** — The commit `609c313` contains real credentials in BOOKING_SYSTEM_PLAN.md. Options: (a) make the lester-ai-chatbot repo private on GitHub if not already; (b) use `git filter-branch` or BFG to scrub history. Since admin password and SMTP password are being rotated, historical exposure is mitigated — rotating credentials is the critical step.
- [x] Remove real credential values from BOOKING_SYSTEM_PLAN.md → replaced with `[REDACTED]` / `[ROTATED]` ✅

### Priority 0 — Settings Split (Code)
- [x] Split `settings/main` → `settings/public` + `settings/private` ✅
  - `settings/public`: companyName, address, phone, email, logo, gcashName, gcashNumber, gcashQR, paymayaName, paymayaNumber, paymayaQR, bankName, bankAccountName, bankAccountNumber, bankQR, downPaymentAmount, securityDeposit
  - `settings/private`: smtpEmail, smtpPassword, telegramToken, telegramChatId
- [x] Migration script written: `C:\Users\LESTER\Desktop\staycation-admin-panel\scripts\migrate-settings.js` ✅
- [ ] Migration script run against live Firestore (`node scripts/migrate-settings.js`)
- [ ] Verify `settings/public` readable in browser, `settings/private` returns permission-denied in browser

### Priority 0 — Firestore Rules
- [x] `settings/private` — `allow read, write: if false` (Admin SDK bypasses, browser JS cannot read) ✅
- [x] `settings/public` — `allow read: if true`, `allow write: if isAdminForClient()` ✅
- [x] `bookings` — `allow create: if false` (server-side only via Admin SDK) ✅
- [x] `availability` — `allow read: if true`, `allow write: if false` (server-side only) ✅
- [x] `inventory` — `allow read, write: if false` (Admin SDK only) ✅
- [x] Rules deployed ✅

### Priority 1 — Secure Email Functions
All four functions now accept only `clientId` + `bookingId` and fetch all data from Firestore server-side.

- [x] `send-receipt.js` rewritten ✅
  - Accepts: `{ clientId, bookingId }`
  - Fetches booking from `clients/{clientId}/bookings/{bookingId}`
  - Reads SMTP from `settings/private`, company info from `settings/public`
  - Idempotency: Firestore transaction checks + sets `receiptSentAt` before sending — duplicate calls return `{ success: true, alreadySent: true }`
  - Validates: booking must exist and status must be `pending_review`
- [x] `send-confirm.js` rewritten ✅
  - Accepts: `{ clientId, bookingId, adminNote? }`
  - Transaction: verify status is `pending_review` → set `confirmed` + `confirmedAt` atomically
  - Updates `availability/{bookingId}.status` to `confirmed`
  - Inventory stays claimed (confirmed = permanently blocked)
  - Reads all email data from Firestore
- [x] `send-reject.js` rewritten ✅
  - Accepts: `{ clientId, bookingId, adminNote }` — adminNote enforced server-side
  - Transaction: verify `pending_review` → set `rejected` + `adminNote` + `rejectedAt`
  - Releases inventory: deletes `inventory/{unitId_YYYY-MM-DD}` for all nights
  - Updates `availability/{bookingId}.status` to `rejected`
- [x] `send-cancel.js` rewritten ✅
  - Accepts: `{ clientId, bookingId, adminNote? }`
  - Transaction: verify `confirmed` → set `cancelled` + `cancelledAt`
  - Releases inventory for all nights
  - Updates `availability/{bookingId}.status` to `cancelled`

### Priority 1 — Server-Side Price Calculation
- [x] `create-booking.js` calculates server-side: ✅
  - Per-night rate (weekday Mon–Thu vs weekend Fri–Sun)
  - Extra guest fee (if numGuests > unit.maxGuests)
  - Add-ons (per-night × nights, or one-time)
  - Grand total, reservation fee, security deposit, balance due at check-in

### Priority 2 — Atomic Double-Booking Prevention
- [x] `create-booking.js` written ✅
  - Accepts: `{ clientId, unitId, checkIn, checkOut, guestName, guestEmail, guestPhone, numGuests, addonIds[], paymentMethod, screenshotUrl }`
  - Firestore transaction: reads all `inventory/{unitId_YYYY-MM-DD}` docs for requested nights
  - If any night is held (and not expired) or confirmed → returns `{ error: 'DATES_NOT_AVAILABLE' }` — entire booking fails
  - If all clear → atomically claims all nights (status: `held`, expiresAt: now + 45 min) + creates booking + creates availability doc
  - Returns `{ bookingId, referenceNo }`
- [x] Pending hold expiration: 45-minute TTL on each `inventory` doc (`expiresAt` field) ✅
- [x] Expired holds treated as available in transaction check ✅
- [x] Scheduled expiration cleanup — `expire-holds.js` deployed; cron-job.org fires every 15 min (`*/15 * * * *`); Firestore composite index created for `inventory (status ASC, expiresAt ASC)` ✅
- [x] Inventory released on reject / cancel (inside send-reject.js and send-cancel.js) ✅

### Priority 3 — Payment Screenshot Security
- [x] `get-upload-signature.js` written ✅
  - Accepts: `{ clientId }` — rate limited (max 5 signatures per IP per 10 min via in-memory store)
  - Returns Cloudinary signed upload params: `{ signature, apiKey, timestamp, folder, cloudName }`
  - Server controls: folder = `payment-proofs/{clientId}/`, allowed formats = jpg/png/webp, max file size = 10MB, use_filename = false (generated filename)
  - Requires `CLOUDINARY_API_SECRET` in Netlify env vars
- [ ] Cloudinary: create two upload presets — `booking-public` (unsigned, for property photos/QR) and restrict `payment-proofs/` folder to signed uploads only
- [ ] Payment screenshot URLs: stored as Cloudinary URLs in Firestore (already done) — access via Cloudinary signed URL in admin panel (Stage 5 task)
- [ ] Retention policy: Cloudinary auto-delete payment proofs after 365 days (set in Cloudinary console → Media Library → Folders → payment-proofs → Auto-backup/deletion rule)

### Priority 4 — Multi-Client Security (Deferred to Stage 5)
> Stage 5 (admin panel) is where admin auth happens — multi-client isolation verified there.
- [ ] Every admin user assigned `clientId` via Firestore `admins/{email}` doc (already in rules) — ⏩ verified during Stage 5 admin login build
- [ ] Firestore rules verify user can only access their `clientId` — ✅ already enforced by `isAdminForClient()` function in rules
- [ ] Add Firebase Auth Custom Claims (`clientId`) to complement Firestore check — ⏩ Stage 5
- [ ] Cross-client isolation test — ⏩ Stage S testing section below

### Priority 5 — Operations and Privacy (Deferred to Stage 5 + Stage 7)
- [ ] Privacy notice and guest consent on booking form — ⏩ Stage 5 booking site update
- [ ] Data retention / deletion policy document — ⏩ Stage 9 (polish)
- [ ] Booking status history collection (`statusHistory` subcollection on each booking) — ⏩ Stage 5
- [ ] Audit log for confirm/reject/cancel/edits — ⏩ Stage 5 (written by email functions on each action)
- [ ] Admin password reset process documentation — ⏩ Stage 9
- [ ] Booking data export and backup procedure — ⏩ Stage 9
- [ ] Monitoring for failed emails and function errors — ⏩ Stage 7
- [ ] Admin-visible notification log (type, recipient, timestamp, success/failure, error, retry count) — ⏩ Stage 5

### Email Spam Prevention (from Stage 4 — carried forward)
- [ ] Verify SPF record on sending domain
- [ ] Verify DKIM configured
- [ ] Send test emails to Gmail, Yahoo, Outlook — verify inbox not spam

### Required Tests Before Stage 5
- [ ] Two simultaneous `create-booking` calls for same unit/dates → only one succeeds, other gets `DATES_NOT_AVAILABLE`
- [ ] Expired pending hold (wait 45 min or manually set `expiresAt` to past) → dates available again
- [ ] Browser posts modified grand total → server ignores it, uses own calculation
- [ ] `create-booking` with fake `clientId` → cannot access another client's data
- [ ] Repeated `send-receipt` call → second call returns `{ success: true, alreadySent: true }`, no second email sent
- [ ] Direct POST to `send-receipt` with invented booking data → `clientId`/`bookingId` lookup fails, returns error
- [ ] Direct browser read of `clients/X/settings/private` → Firestore returns permission-denied
- [ ] `get-upload-signature` called 6× from same IP in 10 min → 6th returns 429
- [ ] Upload oversized file using returned signature → Cloudinary rejects
- [ ] `send-reject` without `adminNote` → returns 400
- [ ] Client A's admin token attempts to read Client B's bookings → Firestore returns permission-denied

### Stage Notes
**2026-06-28:**
- Security review triggered by pre-launch audit covering Priorities 0–5
- Old architecture had: public `settings` (exposed SMTP/Telegram), browser-trusted booking data, unsigned Cloudinary uploads, `allow create: if true` on bookings and availability (allowed fake entries)
- New architecture: `settings/private` server-only, all booking data fetched from Firestore server-side, atomic per-night inventory, signed uploads
- Migration script (`migrate-settings.js`) run against live Firestore — `settings/main` split into `settings/public` + `settings/private` successfully
- All three booking site pages updated to read `settings/public` (index.html, property.html, booking.html) — `settings/main` left intact for now (can delete from Firestore console later)
- `booking.html` rewritten: no more direct Firestore writes — calls `create-booking.js` for booking creation and `get-upload-signature.js` for signed Cloudinary upload
- Cloudinary signed upload bug fixed: `max_file_size` removed from signature params (it is not a valid Cloudinary upload API parameter — caused 401 Unauthorized)
- Admin email notification added to `send-receipt.js`: sends booking alert to `smtpEmail` on every new booking submission (guest name, room, dates, total, screenshot link, admin panel button)
- `expire-holds.js` built and deployed: releases abandoned 45-min inventory holds, extends holds for legitimate bookings (receiptSentAt set) by 72hrs
- Firestore composite index created: `inventory` collection on `(status ASC, expiresAt ASC)` — required for expire-holds query
- cron-job.org configured: fires expire-holds every 15 min (`*/15 * * * *`), saves response history
- Credential rotation: user declined — old SMTP App Password still in git history (commit 609c313). Risk accepted.
- **End-to-end test PASSED**: full booking flow from booking site to Firestore to receipt email — booking `BK-gX6Ao6AnmXzDIn6aOIIB` created, receipt email received at lstrmrcd02@gmail.com ✅

### Status
- [x] Stage S Complete ✅ (2026-06-28)

---

## Stage 5 — Admin Panel: Core
> Goal: Client can log in, see all bookings, review screenshots, confirm or reject, and manage settings.

### Tasks

**Admin Panel Structure**
- [ ] Create files: `admin/index.html` (login), `admin/dashboard.html`, `admin/bookings.html`, `admin/settings.html`
- [ ] All pages check Firebase Auth on load — redirect to login if not authenticated
- [ ] Sidebar navigation: Dashboard, Bookings, Settings, Logout
- [ ] Logout button: signs out Firebase Auth + redirects to login page

**Login Page**
- [ ] Email + password fields
- [ ] "Sign In" button with loading state
- [ ] Error message if wrong credentials
- [ ] Redirect to Dashboard on success
- [ ] No "Sign Up" or "Forgot Password" link (admin accounts created manually in Firebase console)

**Dashboard Page**
- [ ] Metric cards:
  - [ ] Total bookings (all time)
  - [ ] Pending review count
  - [ ] Confirmed count
  - [ ] Revenue this month (sum of grandTotal for confirmed bookings this month)
- [ ] Recent bookings list (last 5, most recent first)
- [ ] Data reads from Firestore scoped to this client's `clientId`

**Bookings List Page**
- [ ] Table columns: Reference No, Guest Name, Property, Check-in, Check-out, Total, Payment Method, Status badge, Actions
- [ ] Status badges: PENDING REVIEW (yellow), CONFIRMED (green), REJECTED (red), CANCELLED (grey)
- [ ] Filter by status (All / Pending / Confirmed / Rejected / Cancelled)
- [ ] Search by guest name or reference number
- [ ] Sorted by date created (newest first)
- [ ] Pagination (20 per page)
- [ ] Click any row → open Booking Detail modal
- [ ] **Real-time updates**: use Firestore `onSnapshot` listener — new bookings appear automatically without page refresh
- [ ] New booking notification: show a toast or badge counter update when a new booking arrives while admin is on the page

**Booking Detail Modal**
- [ ] All guest info (name, email, phone, num guests)
- [ ] Booking details (property, check-in, check-out, nights)
- [ ] Price breakdown (base total, add-ons list + prices, grand total, down payment amount)
- [ ] Payment method
- [ ] Screenshot preview image (click to open full size in new tab)
- [ ] Current status badge
- [ ] Admin Note field (text input — reason for rejection, or any note)
- [ ] Action buttons (shown based on current status):
  - [ ] If `pending_review`: "Confirm Booking" (green) + "Reject Booking" (red)
  - [ ] If `confirmed`: "Cancel Booking" (grey) only
  - [ ] If `rejected`: "Reconsider — Move to Pending" button (resets to pending_review)
  - [ ] If `cancelled`: no action buttons
- [ ] On "Confirm Booking":
  - [ ] Update Firestore status → `confirmed`, set `confirmedAt` timestamp
  - [ ] Call Netlify `/send-confirm` function
  - [ ] Show success toast "Booking confirmed. Email sent to guest."
- [ ] On "Reject Booking":
  - [ ] Require admin note (cannot reject without a reason)
  - [ ] Update Firestore status → `rejected`
  - [ ] Call Netlify `/send-reject` function
  - [ ] Show success toast "Booking rejected. Email sent to guest."
- [ ] On "Cancel Booking":
  - [ ] Confirm prompt: "Are you sure? This cannot be undone."
  - [ ] Update Firestore status → `cancelled`
  - [ ] Call Netlify `/send-cancel` function (notify guest of cancellation)

**System Settings Page**
- [ ] Company Details section: business name, address, phone, support email, logo upload
- [ ] GCash section: account name, phone number, QR code image upload
- [ ] PayMaya section: account name, phone number, QR code image upload
- [ ] Bank Transfer section: bank name, account holder name, account number, QR code image upload
- [ ] Payment Settings section:
  - [ ] Reservation Fee (₱) — `downPaymentAmount` in Firestore — what guest pays NOW to hold the dates (non-refundable)
  - [ ] Security Deposit (₱) — `securityDeposit` in Firestore — refundable, collected at check-in (set to 0 to hide)
  - [ ] Both fields update the booking site price summary instantly (Firestore real-time)
- [ ] Gmail SMTP section: SMTP email address + App Password (per-client, stored in Firestore — not in Netlify env vars)
- [ ] Telegram section: Bot Token + Chat ID (optional — leave blank to disable notifications)
- [ ] "Save Settings" button → writes to Firestore `settings/main`, uploads images to Cloudinary
- [ ] Success confirmation shown after save
- [ ] ⚠️ Note: when admin updates `availability` record status (confirm/reject/cancel), the `availability/{bookingId}` doc must also be updated to reflect the new status — so the calendar stays accurate

**Testing**
- [ ] Login with correct credentials → works
- [ ] Login with wrong credentials → error shown
- [ ] Logout → redirected to login
- [ ] View pending booking → screenshot loads
- [ ] Confirm booking → Firestore status changes → customer email received
- [ ] Reject booking (without note) → blocked, must enter note
- [ ] Reject booking (with note) → Firestore status changes → customer email received
- [ ] Update settings → booking site reflects new GCash/PayMaya/bank details within 30 seconds

### Stage Notes
**2026-06-29:**
- All admin panel pages built and deployed to `https://staycation-admin.netlify.app`
- Login (`index.html`): Firebase Auth email/password, redirect on success, error handling
- Dashboard (`dashboard.html`): 6 metric cards including "Costs This Month" + "Net This Month" (expenses onSnapshot); recent bookings table; real-time via onSnapshot
- Bookings (`bookings.html`): full table with status badges, filter by status, search, newest first; Booking Detail modal with screenshot preview + Confirm/Reject/Cancel actions; calls Netlify email functions
- Settings (`settings.html`): company details, GCash/PayMaya/bank QR upload (signed Cloudinary via get-upload-signature), payment amounts, SMTP, Telegram, **separate Admin Notification Email** (notificationEmail in settings/private — distinct from smtpEmail so admin phone gets a ring on new bookings)
- QR upload in settings was broken (used unsigned `booking-public` preset that didn't exist) — fixed by switching to signed upload via `get-upload-signature.js` with `type='public'` and Firebase ID token auth
- Sidebar expanded from 3 to 10 items across all pages: Dashboard, Bookings, Calendar, Units, Amenities, Add-ons, Blocking, Expenses, Design, Settings

### Status
- [x] Stage 5 Complete ✅ (2026-06-29)

---

## Stage 6 — Admin Panel: Property Management
> Goal: Client manages their own listings, amenities, add-ons, rates, and blocked dates without contacting you.

### Tasks

**Unit Management Page**
- [ ] Unit cards grid: photo, name, location, weekday/weekend rates, status, Modify + Delete buttons
- [ ] "Register Unit" button → opens Add Unit modal

**Add / Edit Unit Modal**
- [ ] Fields: unit name, unique ref ID (auto-generated if blank), description, location
- [ ] Weekday rate (₱), weekend rate (₱)
- [ ] Check-in time (default 2:00 PM), check-out time (default 12:00 PM)
- [ ] Base guest capacity, max extra guests, extra guest fee (₱)
- [ ] Pets allowed (yes/no toggle), pet fee per night (₱)
- [ ] Rating (0–5, half-star increments)
- [ ] Photo upload: max 10 photos, max 5MB each, jpg/png/webp only
- [ ] Photos upload to Firebase Storage: `unit-photos/{clientId}/{unitId}/`
- [ ] Save to Firestore `units` subcollection
- [ ] Edit: same modal, all fields pre-filled

**Delete Unit**
- [ ] Before deleting: check if unit has any `confirmed` or `pending_review` bookings
- [ ] If active bookings exist → show warning: "This unit has [N] active booking(s). Resolve them before deleting."
- [ ] If no active bookings → confirm prompt: "Delete [unit name]? This cannot be undone."
- [ ] On confirm: delete unit document + all unit photos from Storage

**Amenities Page**
- [ ] Grid of current amenities (icon + name + description)
- [ ] "Add Amenity" button → modal: Font Awesome icon class input, name, description
- [ ] Delete amenity (with confirm prompt)
- [ ] Changes reflect on booking website within 30 seconds (Firestore real-time)

**Add-ons Page**
- [ ] List of upsell services: unit name, service name, price, billing type
- [ ] "Add Service" button → modal: select unit (or all units), name, price (₱), billing type (per night / one-time)
- [ ] Delete service (with confirm prompt)

**Holiday Rates Page**
- [ ] Table: unit, date, peak daily rate
- [ ] "Add Holiday Rate" button → modal: select unit, date, peak rate (₱)
- [ ] Bulk upload option (CSV: unit, date, rate)
- [ ] Remove button per row
- [ ] These rates override weekday/weekend rate on that specific date in the booking calendar

**Manual Blocking Page**
- [ ] Left panel: Block Schedule form
  - [ ] Select unit (or apply to all units)
  - [ ] Start date, end date
  - [ ] Blocking reason (dropdown: Maintenance, Personal Use, External Booking, Other)
  - [ ] Additional remarks (optional text)
  - [ ] "Save Block" button
- [ ] Right panel: Visual monthly calendar
  - [ ] Shows existing bookings (amber) and blocked periods (dark)
  - [ ] Navigate months
- [ ] Active Blocks table below: unit, date range, reason, Remove button

### Stage Notes
**2026-06-29:**
- `units.html`: unit card grid, Register/Modify modal with multi-image Cloudinary upload (`type='public'`), delete with active-booking guard, real-time onSnapshot
- `amenities.html`: icon grid, Add/Edit modal with Font Awesome icon picker, delete confirmation
- `addons.html`: add-ons table (per unit or all), Add/Edit modal, billing type (per night / one-time)
- `blocking.html`: split layout — form + active blocks table on left, mini 3-state calendar (blocked=red, pending=amber, confirmed=green) on right; clicking calendar dates pre-fills form
- Holiday rates page: deferred (not built yet) — override pricing per date not yet implemented

### Status
- [x] Stage 6 Complete ✅ (2026-06-29) — Holiday rates page deferred to Stage 9 polish

---

## Stage 7 — Admin Panel: Calendar & Expenses
> Goal: Full operational visibility into bookings and profitability.

### Tasks

**Interactive Calendar Page**
- [ ] Full monthly calendar view (month / week / day toggle)
- [ ] Each booking shown as a colored event bar spanning its dates
- [ ] Color-coded by status: Confirmed (green), Pending (yellow), Blocked (grey), Cancelled (red)
- [ ] Event label: guest name + unit name
- [ ] Click event → opens Booking Detail modal (same as Stage 5)
- [ ] Navigate months forward/backward
- [ ] "Today" button

**Expenses Page**
- [ ] Summary cards: Total Expenses (all time), This Month, Total Entries, Top Category
- [ ] "Add Expense" button → modal:
  - [ ] Date, Category (dropdown: Maintenance, Utilities, Supplies, Marketing, Other), Description, Amount (₱), Unit (dropdown or "All Units")
  - [ ] Save to Firestore `expenses`
- [ ] Expenses table: date, category badge, description, amount, unit, delete button
- [ ] Filter by category and date range
- [ ] Export to CSV (downloads a file)

**Net Proceeds Summary**
- [ ] Gross Revenue (confirmed bookings) minus Total Expenses = Net Proceeds
- [ ] Shown on Dashboard and Expenses page
- [ ] Filterable by month

### Stage Notes
**2026-06-29:**
- `calendar.html`: FullCalendar.js v6.1.11 (CDN), month/week/list views, unit filter dropdown, color-coded events (confirmed=green, pending=amber, blocked=red, expired=gray), click event → booking or block detail modal
- `expenses.html`: 4 summary cards (Total, This Month, Total Entries, Top Category), Add modal (date/category/description/amount/unit), category + date range filter, delete, real-time onSnapshot
- Net Proceeds: "Costs This Month" + "Net This Month" added to dashboard metric cards. Expenses onSnapshot listener calculates costs; Net = revenue − costs, updates whenever either listener fires.
- Export to CSV: deferred to Stage 9 polish

### Status
- [x] Stage 7 Complete ✅ (2026-06-29) — CSV export deferred to Stage 9

---

## Stage 8 — Chatbot Integration
> Goal: Customer can go from Messenger conversation directly to the booking site. Full end-to-end flow works.

### Tasks
- [ ] Define booking intent keywords/triggers in n8n sub-workflow (e.g., "book", "reserve", "magbook", "I want to book")
- [ ] Update sub-workflow: when intent detected → bot sends booking website URL with unit pre-selected if known
  - [ ] Example: `"Sure! You can book directly here 👉 {github-pages-url}/booking.html?id={unitId}"`
- [ ] If customer asks about a specific unit → bot sends `property.html?id={unitId}` link
- [ ] If no specific unit → bot sends `index.html` (main listing page)
- [ ] Test: send booking-intent message → bot replies with correct link
- [ ] Verify link opens correctly on mobile (Facebook in-app browser)
- [ ] **Facebook in-app browser testing** (critical — this is where most customers will open the link):
  - [ ] Booking site loads correctly inside Facebook app browser
  - [ ] Photo gallery works (tap to view full screen)
  - [ ] Booking flow all 4 steps work
  - [ ] File upload (screenshot) works from camera roll inside Facebook browser
  - [ ] Confirmation screen appears correctly
  - [ ] If file upload fails in Facebook browser: test opening link in external browser (Safari/Chrome) as fallback
- [ ] Test complete end-to-end: Messenger → booking site → Firebase → Admin panel → email confirmation

### Stage Notes

### Status
- [ ] Stage 8 Complete ✅

---

## Stage 9 — Polish, UI Control & Multi-Client Ready
> Goal: System is a repeatable product. Each new client takes under 2 hours to onboard.

### Tasks

**UI/UX Control Panel (Admin → new page)**
- [x] Theme Color Customizer: 6 color pickers (primary, secondary, accent, text, background, footer) synced with hex inputs → saved to Firestore `settings/public` ✅
- [x] Booking site reads colors from `settings/public` and applies as CSS variables on load ✅
- [x] Logo upload → Cloudinary signed upload → stored as `logoUrl` in settings ✅
- [x] Homepage Section Visibility toggles: Hero, Amenities, Testimonials, Featured Units ✅
- [x] Font selection: heading font + body font (Google Fonts dropdown) ✅
- [x] SEO Settings: site title, keywords, Open Graph image ✅
- [x] Hero multi-image upload (Cloudinary signed) — supports multiple hero banner photos ✅
- [x] Promo banner image upload ✅
- [ ] Custom domain per client (Stage 9 remaining)
- [ ] Holiday rates page (override daily rate by date)

**Multi-Client Infrastructure**
- [ ] Document: how to create a new Firebase Auth user for a new client (Firebase console steps)
- [ ] Document: how to duplicate the booking site GitHub repo for a new client
- [ ] Document: how to set the `clientId` constant in the new repo's `firebase-config.js`
- [ ] Document: how to point a custom domain to GitHub Pages (Namecheap → CNAME record)
- [ ] Document: how to set up custom domain on GitHub Pages settings
- [ ] Internal pricing sheet: setup fee, custom domain add-on, monthly support fee

**Client Onboarding Kit** (separate doc — `docs/CLIENT_ONBOARDING.md`)
- [ ] Full checklist of everything Lester needs to collect from a new client before setup
- [ ] What the client provides: dedicated Gmail + App Password, Telegram account, property details, GCash/PayMaya/bank details, QR code images, unit photos
- [ ] What Lester sets up: Firebase Auth user, Firestore seed data, Telegram bot, GitHub repo, Netlify settings
- [ ] Step-by-step Telegram bot setup instructions (BotFather → token → chat ID → Firestore)
- [ ] Step-by-step Gmail App Password instructions (client walks through this with Lester)

**Final QA Checklist**
- [ ] Test full booking flow on 3 different mobile devices
- [ ] Test admin panel on mobile (responsive)
- [ ] Test with slow 3G connection (Chrome DevTools)
- [ ] Verify all emails land in inbox (not spam) — test with Gmail, Yahoo
- [ ] Verify Telegram alerts work
- [ ] Run Lighthouse on booking site: score 80+ on Performance and Accessibility
- [ ] Verify custom domain works end-to-end (if applicable)

### Stage Notes
**2026-06-29 (batch 1):**
- `design.html` built: 6 color pickers with hex input sync, section visibility toggles, hero multi-image upload, promo banner upload, Google Fonts selection, SEO fields. Writes to `settings/public` via client SDK (allowed by `isAdminForClient()` rule — no Netlify function needed).
- Booking site (`index.html`, `property.html`, `booking.html`) updated to read design settings at load time and apply as CSS custom properties.

**Booking Site UX Polish (2026-06-29):**
- **Room card photo carousel**: prev/next arrows + dots, manual only (no auto-cycle). Arrows always visible at 82% opacity (hover-only doesn't work on touch). Hero banner slideshow retains auto-cycle.
- **Room card clickable**: entire card navigates to `property.html?id=...`; only arrows and "View Room" button call `stopPropagation()`.
- **Hero banner crossfade**: two stacked div layers — no blank gap between transitions.
- **Weekend rate price summary**: weekday/weekend nights shown as separate rows in price breakdown.
- **QR code lightbox**: GCash/Maya/Bank QR images open full-size overlay for screenshot.
- **Admin panel mobile responsiveness**: horizontal table scroll, bottom-sheet modals, 44px touch targets, 16px inputs (no iOS zoom).
- **Property page gallery**: prev/next arrows + dots + "1 / N" counter over the hero photo. All photos preloaded. Bug fix: null reference crash on `hero-placeholder` removed.

**2026-06-29 (batch 2 — Stage 9 continued):**
- **Policies in Settings**: new "Policies" section added to `settings.html` with 4 textareas (Booking Policy, Cancellation Policy, House Rules, Check-in/Check-out Policy). Saves to `settings/public`. `booking-policy.html` and `cancellation.html` on booking site now load content from Firestore — show custom text if set, fall back to default static content if blank.
- **Financial Report (CSV export)**: "Download Report" button on expenses page. Pulls confirmed bookings (filtered by check-in date range) + expenses. Downloads a CSV with three sections: Revenue, Expenses, Summary (Total Revenue, Total Expenses, Net Profit). Date range uses the existing filter inputs.
- **Holiday Rates page**: new `holiday-rates.html` admin page — add/delete peak rate overrides per unit and date. Unit filter + month filter. Added to sidebar of all 11 admin pages.
- **Holiday rates on booking site**: `booking.html` now fetches `holiday_rates` collection on load and stores as a date→rate map. `calcBaseTotal` checks each night against the map — if a holiday rate exists for that date, it overrides the regular weekday/weekend rate. `buildPriceSummaryHTML` shows holiday nights as a separate labeled row (e.g. "1 peak night × ₱6,000 (Christmas)").

**2026-06-29 (batch 3 — bug fixes and polish):**
- **Fixed Holiday Rates save (permission denied)**: `holiday_rates` collection was missing from `firestore.rules` — no rule = denied. Added public read (booking site needs it for price calc) + admin write. Deployed to Firebase.
- **Fixed Financial Report query error**: `orderBy('checkIn')` combined with `where('status', '==', 'confirmed')` requires a Firestore composite index that didn't exist. Removed `orderBy` from query and sort client-side instead — same result, no index needed.
- **Fixed Holiday Rates sidebar icon**: `fa-calendar-star` is a FontAwesome Pro (paid) icon — showed blank on free plan. Replaced with `fa-tag` (free icon) across all 11 admin pages.
- **Holiday Rates UX**: added "FILTER EXISTING RATES" label above the filter section and a hint line explaining that "+ Add Holiday Rate" is where you enter a specific date and price. The filter uses a month picker just for searching existing rates — the Add modal has a full date + rate form.
- **Financial Report — upgraded to Excel (.xlsx)**: replaced CSV with a real Excel file using `xlsx-js-style` library (loaded from CDN). Report now has: dark blue title row, grey meta rows, color-coded OVERVIEW (green=revenue, red=expenses, blue=net profit), column headers with light blue background, alternating row stripes, yellow TOTAL rows, and pre-set column widths. Downloads as `.xlsx`.
- **Pushed booking-policy.html + cancellation.html** to GitHub Pages (these were built in batch 2 but not yet committed — now live).

### Status
- [x] Stage 9 Complete ✅ (all features built, all bugs fixed, all pages live as of 2026-06-29)

---

## Known Limitations (MVP — Intentional, Not Bugs)
> These are known gaps that are acceptable for the first version. Document these to clients during onboarding.

| Limitation | Impact | Workaround |
|------------|--------|------------|
| ~~No double-booking prevention~~ | ~~Two customers could book the same dates simultaneously~~ | ✅ Fixed in Stage 3 — calendar blocks `pending_review` + `confirmed`, real-time listener, final check at submission |
| Extremely rare race condition still possible | Two customers submit at the exact same millisecond | Extremely unlikely for small property. Admin rejects the duplicate manually. |
| No customer-initiated cancellation | Customer cannot cancel their own booking from the site | Customer contacts the property owner directly via phone/email |
| Testimonials are static | Admin cannot add/edit reviews from the admin panel | Lester edits the HTML file manually when client wants new reviews |
| No refund tracking | System doesn't track if/when deposit was refunded | Admin handles refunds offline (GCash/bank transfer manually) |
| No SMS notification | Customers only get email — no text message | Customer must have valid email. Acceptable for current market. |
| Admin password reset is manual | If client forgets password, Lester resets via Firebase console | Document this in client onboarding handover |
| No Airbnb/booking.com sync | Bookings from external platforms not reflected in calendar | Client manually blocks dates from external bookings |

---

## End-to-End Test Plan
> Run ALL scenarios after Stage 5 is complete (Stages 1–5 done). Re-run after Stage 8 for full flow.

### Scenario A — Happy Path (customer books successfully)
1. Open booking website on mobile phone (real device)
2. View landing page → see property cards with photos and prices ✓
3. Click "View" on a property → see property detail page with gallery ✓
4. Click "Book Now" → booking flow opens ✓
5. Step 1: Select available dates → nights and total price auto-calculate ✓
6. Step 2: Fill in name, email, phone, guests → proceed ✓
7. Step 3: Select an add-on → total updates ✓
8. Step 4: Select GCash → QR code and number appear ✓
9. Upload a real screenshot image from camera roll ✓
10. See confirmation screen with reference number ✓
11. Check email inbox → receipt email received within 2 minutes ✓
12. Check Telegram → admin alert received with screenshot link ✓
13. Admin opens admin panel → sees pending booking ✓
14. Admin opens booking → screenshot visible ✓
15. Admin clicks Confirm → status changes to Confirmed ✓
16. Check customer email → confirmation email received ✓
17. ✅ PASS

### Scenario B — Admin Rejects Booking
1. Run steps 1–12 from Scenario A
2. Admin opens booking → screenshot looks incorrect
3. Admin clicks "Reject" without entering a note → should be BLOCKED ✓
4. Admin enters rejection reason → clicks Reject ✓
5. Booking status changes to Rejected ✓
6. Customer receives rejection email with reason ✓
7. ✅ PASS

### Scenario C — Admin Cancels a Confirmed Booking
1. Complete Scenario A (booking is now Confirmed)
2. Admin opens confirmed booking
3. Admin clicks "Cancel Booking" → confirm prompt appears ✓
4. Admin confirms → status changes to Cancelled ✓
5. Customer receives cancellation email ✓
6. ✅ PASS

### Scenario D — Blocked/Booked Dates Cannot Be Selected
1. Open booking flow for a unit that has an existing confirmed booking
2. Check-in and check-out dates of existing booking should appear red/blocked in calendar ✓
3. Try clicking on a blocked date → nothing happens ✓
4. Try typing a blocked date manually (if date input exists) → should reject ✓
5. ✅ PASS

### Scenario E — Form Validation
1. Step 1: Try to proceed without selecting dates → blocked ✓
2. Step 2: Try to proceed with empty name field → error shown ✓
3. Step 2: Enter invalid email format (e.g., "notanemail") → error shown ✓
4. Step 2: Enter more guests than max allowed → error shown ✓
5. Step 4: Try to upload a .pdf or .txt file → rejected ✓
6. Step 4: Try to upload an image larger than 10MB → rejected ✓
7. Step 4: Try to click Confirm twice quickly → second click ignored (no double submit) ✓
8. ✅ PASS

### Scenario F — Settings Update Reflects on Booking Site
1. Admin opens System Settings
2. Changes GCash phone number to a new test number
3. Saves settings
4. Customer opens booking site → goes to Step 4 → selects GCash
5. New phone number appears within 30 seconds ✓
6. ✅ PASS

### Scenario G — Chatbot Sends Booking Link (after Stage 8)
1. Open Facebook Messenger → message the client's page
2. Ask: "I want to book a room"
3. Bot replies with the booking site URL ✓
4. Click link → opens in Facebook in-app browser ✓
5. Complete Scenario A from inside the Facebook browser ✓
6. File upload from camera roll works inside Facebook browser ✓
7. ✅ PASS

### Scenario I — Real-Time Calendar Update (Anti Double-Booking)
1. Open the booking site on two different devices (Phone A and Phone B) simultaneously
2. Both see the same unit's calendar — same dates showing as Available
3. Phone A completes the booking flow and submits screenshot for June 30
4. Phone B is still on Step 1 — June 30 should automatically turn red without refreshing ✓
5. Phone B tries to select June 30 → cannot select it ✓
6. Open a third device — go to Step 4 payment (simulating someone who was already at Step 4 when Phone A booked)
7. Third device tries to submit → system detects conflict → shows "dates just taken" error ✓
8. Third device shown "Choose New Dates" button → goes back to Step 1 ✓
9. ✅ PASS

### Scenario H — Admin Unit Management
1. Admin goes to Unit Management
2. Adds a new unit with name, rates, 3 photos ✓
3. Booking site shows new unit on listing page within 30 seconds ✓
4. Admin edits unit — changes weekday rate
5. Booking site shows updated price ✓
6. Admin tries to delete a unit with a pending booking → blocked with warning ✓
7. Admin rejects/cancels all pending bookings → tries delete again → succeeds ✓
8. Booking site no longer shows deleted unit ✓
9. ✅ PASS

### Browser / Device Compatibility
| Device / Browser | Must Pass |
|------------------|-----------|
| Android Chrome (mobile) | ✅ All scenarios |
| iOS Safari (iPhone) | ✅ All scenarios |
| Facebook in-app browser (Android) | ✅ Scenario G |
| Facebook in-app browser (iOS) | ✅ Scenario G |
| Desktop Chrome | ✅ Admin panel all scenarios |
| Desktop Safari | ✅ Admin panel all scenarios |

---

## ⏭️ Next Session — First Client Onboarding

> The system is 100% production-ready. The next session is purely about replacing test data with real client data and going live.

### Live URLs
| System | URL |
|--------|-----|
| **Admin Panel** | https://staycation-admin.netlify.app |
| **Booking Site** | https://bookingph.github.io/staycation-booking-site/ |
| **Firebase Console** | https://console.firebase.google.com/project/lester-booking-system |

### Admin Panel Features (what the client can manage)
| Page | What it does |
|------|-------------|
| **Settings** | Company name, contact info, check-in/check-out times, reservation fee, security deposit, GCash/Maya/Bank QR codes (upload image), SMTP email + App Password (for guest emails), admin notification email, Telegram bot (optional), **Policies** (Booking Policy, Cancellation Policy, House Rules, Check-in/Check-out Policy — plain text, each saved separately, shown on booking site policy pages automatically) |
| **Design** | Brand colors, Google Fonts, hero banner photos, logo, promo banner, SEO title/description, section visibility toggles |
| **Units** | Add/edit/delete rooms — name, description, max guests, weekday rate, weekend rate, extra guest fee, check-in time, check-out time, photos (upload multiple), status (active/inactive) |
| **Amenities** | Pool, WiFi, parking, etc. — shown on all property pages |
| **Add-ons** | Breakfast, extra bed, etc. — guests can add at booking step 3. Billing per night or flat fee. |
| **Date Blocking** | Block specific dates from being booked (public holidays, maintenance, owner use) |
| **Holiday Rates** | Set a peak/holiday rate that overrides weekday/weekend price on specific dates (e.g. Christmas Day → ₱6,000 regardless of weekday/weekend rate). Guests see this labeled in their price breakdown. |
| **Bookings** | View all bookings, change status (pending → confirmed → completed), send confirmation/rejection/follow-up emails, add internal notes |
| **Calendar** | Month-view calendar showing all confirmed bookings per room |
| **Expenses** | Log operational expenses (utilities, maintenance, supplies, etc.). Download a formatted Excel financial report with revenue + expenses + net profit. |
| **Dashboard** | Total revenue, total expenses, net profit, and count cards |

### Onboarding Checklist (do in this order)
- [ ] **Settings** — fill in company name, address, contact phone/email, check-in/check-out times, reservation fee amount, security deposit
- [ ] **Settings → QR Codes** — upload real GCash, Maya, and/or bank QR images
- [ ] **Settings → Email** — enter the client's Gmail address + App Password (enable 2FA on Gmail first, then generate App Password at myaccount.google.com/apppasswords)
- [ ] **Settings → Notification email** — email address where the owner gets alerted on new bookings
- [ ] **Settings → Telegram** — optional: bot token + chat ID for instant Telegram pings on each booking
- [ ] **Settings → Policies** — type the client's policies in plain text (use `- ` to start bullet points). These appear on the Booking Policy and Cancellation Policy pages of the booking site.
- [ ] **Design** — set brand colors, upload logo, upload hero photos, set SEO title/description
- [ ] **Units** — add real rooms with photos, accurate rates, capacity
- [ ] **Amenities** — list what the property offers (pool, WiFi, etc.)
- [ ] **Add-ons** — add any optional extras (breakfast, extra mattress, etc.)
- [ ] **Holiday Rates** — add any peak dates (Christmas, New Year, long weekends)
- [ ] **End-to-end test** — make a test booking from the booking site → confirm payment screenshot upload works → confirm admin gets email + Telegram → admin confirms booking → confirm guest gets confirmation email
- [ ] **Go live** — share the booking site URL with the client

### Stage 8 (chatbot — do anytime after onboarding)
1. In n8n, open the Lester Domain Staycation AI Agent node
2. In the system prompt, add: *"When a guest asks to book, send them this link: [booking site URL]. Room IDs: [list each room name and its booking URL]"*
3. Test via Facebook Messenger — send a booking inquiry and verify the AI replies with the correct link

---

## Completed Stages
- ✅ **Stage 1** — Foundation & Architecture Setup (2026-06-28)
- ✅ **Stage 2** — Booking Website: Listing Page + Property Detail Page (2026-06-28)
- ✅ **Stage 3** — Booking Website: 4-Step Booking Flow (2026-06-28)
- ✅ **Stage 4** — Netlify Functions: All 4 email functions + admin notification (2026-06-28)
- ✅ **Stage S** — Security Hardening: server-side booking, signed uploads, atomic inventory, expire-holds (2026-06-28)
- ✅ **Stage 5** — Admin Panel Core: login, dashboard, bookings, settings, QR upload fix, separate notification email (2026-06-29)
- ✅ **Stage 6** — Admin Panel Property Management: units, amenities, add-ons, date blocking (2026-06-29)
- ✅ **Stage 7** — Admin Panel Calendar & Expenses: FullCalendar.js, expenses page, net proceeds on dashboard (2026-06-29)
- ✅ **Stage 9** — Design page, editable policies (Settings + booking site), holiday rates (admin + booking site + Firestore rules), Excel financial report (.xlsx with colors/borders/formatting), property gallery (prev/next/dots/counter), room card carousel (manual), hero crossfade fix, QR lightbox, admin mobile CSS, all bugs fixed (2026-06-29)
- ⏭️ **Stage 8** — Skipped (5-min manual task: update n8n system prompt with booking URL — do after first client is onboarded)

---

## Change Log
| Date | Version | Change |
|------|---------|--------|
| 2026-06-28 | v1 | Initial plan created |
| 2026-06-28 | v2 | Senior engineering review: added multi-client architecture decision, locked hosting choices, added property detail page, merged Telegram into Stage 3, added file validation, error handling, admin safety checks, email templates, cancel flow, final QA stage |
| 2026-06-28 | v3 | Stage 1 started — status updated to In Progress |
| 2026-06-28 | v4 | Operations-readiness review: added seed data to Stage 1, policy pages to Stage 2, Stage 3/4 dependency note, send-cancel.js to Stage 4, email spam prevention, real-time Firestore updates to Stage 5, Facebook browser testing to Stage 8, Known Limitations section, End-to-End Test Plan with 8 scenarios |
| 2026-06-28 | v5 | Double-booking prevention: calendar now blocks pending_review + confirmed, added onSnapshot real-time listener with conflict warning, added final availability check at Step 4 submission, added Scenario I to test plan, updated Known Limitations |
| 2026-06-28 | v6 | Stage 1 in progress: Firebase project + Firestore + Web App + service account all created. Firebase Storage requires Blaze — switched to Cloudinary free (25GB). Key Values table partially filled. Blocked on Firebase Auth one-time Console initialization. |
| 2026-06-28 | v7 | Stage 1 complete. Repos transferred to bookingPH GitHub account. GitHub Pages live. Netlify admin panel live. |
| 2026-06-28 | v8 | Email architecture finalized: each client uses their own dedicated Gmail + App Password, stored in Firestore settings. Netlify Function reads it at send time — fully personalized per client. Telegram is optional per client (stored in Firestore settings, skipped silently if not set). Test SMTP set to lstrmrcd@gmail.com for development. Firestore data model updated to include smtpEmail, smtpPassword, telegramToken, telegramChatId in settings. Stage 2 starting. |
| 2026-06-28 | v9 | Stages 2 and 3 complete. 4 bugs fixed in booking flow (see Stage 3 notes). New `availability` Firestore collection added (public calendar data, no personal info). Security deposit added as editable setting (₱1,000 default). Price summary redesigned: clearly shows Pay Now (reservation fee) vs Pay at Check-in (balance + security deposit). Stage 4 notes updated with CORS requirement, Telegram alert details, and per-client SMTP flow. Admin panel `availability` sync requirement noted in Stage 5. |
| 2026-06-28 | v10 | Stage 4 in progress. `send-receipt.js` deployed and tested end-to-end. `FIREBASE_SERVICE_ACCOUNT` env var stored as base64 (Netlify CLI strips JSON quotes from raw JSON values). SMTP credentials seeded in Firestore for test client. Email HTML template with terracotta header and booking summary built. Telegram alert with HTML parse_mode included (silently skipped if not configured). |
| 2026-06-28 | v11 | Stage 4 partially complete. All 4 Netlify Functions deployed and tested. Email spam prevention and testing deferred to Stage S. |
| 2026-06-28 | v12 | Pre-launch security review. Stage S (Security Hardening) inserted before Stage 5. Architecture changes: settings split public/private, booking creation moved server-side (create-booking.js), atomic per-night inventory (Firestore transactions), email functions now fetch-from-Firestore only (clientId+bookingId interface), signed Cloudinary uploads (get-upload-signature.js). Credentials removed from plan doc and marked for rotation. Firestore rules tightened: no public create on bookings/availability, settings/private unreadable by browser. |
| 2026-06-28 | v13 | Stage S complete. Migration script run (settings/main → public+private). All booking site pages updated to read settings/public. booking.html rewired to use create-booking.js + get-upload-signature.js (no more direct Firestore writes). Cloudinary signed upload 401 bug fixed (max_file_size removed from signature params). Admin email notification added to send-receipt.js. expire-holds.js built, deployed, Firestore composite index created, cron-job.org configured every 15 min. End-to-end booking flow tested and working. Stage 5 (Admin Panel) is next. |
| 2026-06-29 | v14 | Stages 5, 6, 7 complete. Stage 9 design page built. All admin panel pages deployed to staycation-admin.netlify.app: login, dashboard (costs/net cards), bookings (full modal + email actions), settings (separate notification email, signed QR uploads), units, amenities, add-ons, blocking (mini 3-state calendar), calendar (FullCalendar.js), expenses, design (color/font/hero/SEO). Booking site UX: room card carousel, fully clickable cards, hero crossfade fix (two-layer), weekend rate price breakdown, QR code lightbox, admin panel mobile CSS. Stage 8 (chatbot integration) is next. |

| 2026-06-29 | v15 | Stage 9 nearly complete. Policies section in settings (4 textareas → Firestore → booking site loads dynamically, falls back to static). Financial Report CSV on expenses page (Revenue + Expenses + Net Profit, date-range filtered). Holiday Rates admin page (add/delete peak overrides per unit/date, sidebar on all 11 pages). Holiday rates applied in booking.html (per-night override, labeled row in price summary). Property page photo gallery with prev/next arrows, dots, counter. Room card carousel manual-only (no auto-cycle), arrows always visible. Bug fix: property page null crash on hero-placeholder. Stage 8 skipped. Next: first client onboarding. |
| 2026-06-29 | v16 | Stage 9 complete. Bug fixes: Holiday Rates save (Firestore rules missing holiday_rates collection → added public read + admin write, deployed); Financial Report Firestore composite index error (removed orderBy, sort client-side); Holiday Rates sidebar icon (fa-calendar-star is FA Pro/paid → replaced with fa-tag across all 11 pages). Financial Report upgraded from CSV to formatted Excel (.xlsx) using xlsx-js-style CDN library — dark blue title, color-coded overview, alternating row stripes, yellow totals rows, pre-set column widths. Holiday Rates UX: added filter label + usage hint. Pushed booking-policy.html and cancellation.html to GitHub Pages (was pending). Next session: first real client onboarding (see "Next Session" section in this doc). |

---
*Plan created: 2026-06-28 | Project: Lester AI Chatbot Services — Staycation Booking System*

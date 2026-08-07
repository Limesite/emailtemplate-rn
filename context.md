# rates.now — Email Campaign Project Context

> **Last updated:** 2026-08-08
> **Purpose:** Project memory and decision log. Read this first when resuming work.

---

## Project Overview

Campaign emails for **rates.now**, a mortgage rate monitoring service. Leads are acquired via a funnel form where users enter their email and set a target mortgage rate. The email series launches after form submission.

---

## Brand Identity (from funnel screenshot)

| Element | Value |
|---|---|
| Product name | `rates.now` |
| Wordmark style | lowercase, period as brand accent (e.g. `rates.now`) |
| Primary accent color | `#4f46e5` (indigo/purple — used on period in logo, CTAs, links) |
| Background | `#eef2f7` (light blue-gray page bg) |
| Card background | `#ffffff` |
| Headline color | `#0f172a` (near-black) |
| Body text | `#64748b` (slate gray) |
| Muted / labels | `#94a3b8` |
| Success green | `#22c55e` / bg `#f0fdf4` / border `#bbf7d0` |
| Alert blue | `#3730a3` / bg `#eef2ff` / border `#c7d2fe` |
| Font | System stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif` |

### Core UI Patterns (from funnel)
- Rate cards: white bg, `#e2e8f0` border, `14px` border-radius, uppercase label, large bold number
- Lender card: logo icon | name + NMLS | rate / APR / points stats
- Green status banner: "Your target rate is available right now — it's a great time to compare offers."
- Blue monitoring banner: used when rate not yet matched
- CTA block: indigo gradient, white button
- Legal consent: two checkboxes (SMS + TCPA)

---

## Product Logic

- User enters email + sets a **target mortgage rate** in the funnel
- rates.now monitors lenders in real time
- When a lender hits the target, an alert fires (email + SMS)
- Users verify phone to unlock personalized lender quotes
- Platform is free; monetized via lender referrals
- **Not a lender** — comparison/alert platform only

---

## Email Files

### 1. `intro-nurture-email.html`
**Type:** Trigger email — fires immediately after form submission
**Goal:** (1) Confirm alert is active, (2) educate on the service, (3) drive phone verification

**Structure:**
1. Preheader text (hidden preview)
2. Top nav — rates.now wordmark + dashboard link
3. Main card (white, rounded, shadow):
   - Status badge ("Alert Active")
   - Hero headline: `{{first_name}}, we're watching rates for you`
   - Rate cards grid: YOUR TARGET RATE | BEST RATE TODAY
   - Status banner (blue monitoring / green available — conditional)
   - Today's lender rates section (Reliant Home Funding example card)
   - CTA card (indigo gradient): "Get My Personalized Quotes →"
   - "What happens next" — 3-step education
4. Social proof bar (50K+ alerts / 40+ lenders / Free always)
5. Footer (address, privacy/terms/unsubscribe links, legal disclaimer)

---

## Merge Tags / Personalization Variables

| Tag | Description | Example |
|---|---|---|
| `{{first_name}}` | Lead's first name | `Jack` |
| `{{target_rate}}` | Their chosen target rate | `5.5%` |
| `{{best_rate_today}}` | Live best rate at send time | `5.5%` |
| `{{dashboard_url}}` | Link to their rate dashboard | `https://rates.now/dashboard/abc123` |
| `{{verify_phone_url}}` | Phone verification page | `https://rates.now/verify/abc123` |
| `{{signup_date}}` | Date they submitted the form | `August 5, 2026` |
| `{{privacy_url}}` | Privacy policy link | `https://rates.now/privacy` |
| `{{terms_url}}` | Terms of service link | `https://rates.now/terms` |
| `{{unsubscribe_url}}` | One-click unsubscribe | `https://rates.now/unsubscribe/token` |

---

## Technical Decisions

### Mobile-Only Approach
- **Decision:** Email is designed for mobile only (max-width 600px, optimized for 375–430px)
- **Rationale:** User specified mobile-only; funnel is mobile-first
- **Implementation:** `@media only screen and (max-width: 600px)` overrides, table-based layout for email client compatibility

### No External Assets
- No web fonts (Google Fonts, etc.) — system font stack only
- No external images — emoji used for lender icon placeholder (replace with hosted image in production)
- No external CSS files — all styles inline or in `<style>` block
- **Rationale:** Faster load, no render-blocking, works in Gmail which strips `<head>` CSS (mitigated by inline styles on all critical elements)

### Table-Based Layout
- Outer shell uses HTML tables for email client compatibility
- Inner rate card grid uses table cells for side-by-side cards
- Inline styles on every element as fallback
- `<style>` block in `<head>` handles mobile breakpoints

### Conditional Banners
- Blue banner = monitoring (default state for this intro email)
- Green banner = target rate available (commented out in HTML, ready to activate)
- **ESP logic needed:** Dynamic content block or separate template version based on `rate_available` boolean variable

### CTA Strategy
- Primary CTA is phone verification (highest-value action)
- Dashboard link in nav as secondary action
- No secondary CTA buttons to avoid decision paralysis

---

## Compliance Notes

- Lender NMLS #292473 (Reliant Home Funding) shown as example — must be dynamic in production
- Footer disclaimer must include: rates.now not a lender, NMLS disclosures, message/data rates, STOP to unsubscribe
- TCPA language required for any contact consent (already in funnel form checkboxes — email reinforces)
- Must include physical mailing address (CAN-SPAM)
- One-click unsubscribe required

---

## Lender Data (Static Example — Make Dynamic in Production)

| Field | Value |
|---|---|
| Name | Reliant Home Funding |
| NMLS | #292473 |
| Rate | 5.5% |
| APR | 5.78% |
| Points | 1.994 |

In production, pull live lender data from the same API that powers the funnel dashboard.

---

## Next Emails to Build (Planned)

| # | Email | Trigger | Goal |
|---|---|---|---|
| 1 | `intro-nurture-email.html` ✅ | Immediate post-signup | Confirm alert, educate, drive phone verify |
| 2 | `rate-alert-email.html` | When target rate is hit | Alert, show lender match, drive quote request |
| 3 | `follow-up-day3.html` | 3 days if no phone verify | Soft nudge, explain benefits again |
| 4 | `rate-drop-nudge.html` | Rate drops 0.25%+ from last shown | Urgency nudge, compare lenders |
| 5 | `weekly-digest.html` | Weekly (if no conversion) | Rate market summary, re-engage |

---

## ESP / Platform Notes

- Merge tag format: `{{tag_name}}` — adjust to match your ESP (e.g., `*|FNAME|*` for Mailchimp, `{{contact.first_name}}` for HubSpot, `%FIRST_NAME%` for Klaviyo)
- Conditional content blocks needed for: banner color (monitoring vs. available), lender card count
- Suggested subject lines:
  - `{{first_name}}, your rate alert is live 🔔`
  - `We're watching {{target_rate}} for you, {{first_name}}`
  - `Your mortgage rate monitor is active`
- Preview text: `Your rate alert is active. We'll notify you the moment a lender hits your target — here's what's happening right now.`

---

## Open Questions / TODOs

- [ ] Replace lender icon placeholder emoji with hosted lender logo images
- [ ] Confirm physical address for CAN-SPAM footer
- [ ] Decide: single template with dynamic content vs. separate HTML files per state
- [ ] Set up ESP conditional logic for green vs. blue banner
- [ ] A/B test subject lines
- [ ] Add UTM parameters to all links for tracking
- [ ] QA test in: iOS Mail, Gmail app (iOS), Gmail app (Android), Samsung Mail

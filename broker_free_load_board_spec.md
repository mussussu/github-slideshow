# Broker-Free Load Board MVP Specification

## Product Vision
Create a minimalist, Craigslist-inspired load board that connects verified shippers directly with compliant motor carriers without acting as a broker, handling funds, or generating dispatch paperwork. The product focuses on transparency, trust, and quick discovery of available freight.

## User Roles
- **Shipper** – Posts loads after verifying company credentials and attesting they are not operating as a broker.
- **Carrier/Trucker** – Searches and filters listings, saves preferences, and contacts shippers after DOT/MC verification.

## Key Flows
1. **Account Creation & Verification**
   - Email + password signup with email one-time passcode verification.
   - Shippers provide company name, business email domain, business address, EIN (stored securely), and explicit attestation of non-broker status on the posting form.
   - Carriers provide DOT and MC numbers, company name, and phone number; system cross-checks via public safety APIs.
   - Optional phone verification for both roles (phase 2).
2. **Shipper Posting**
   - Form fields: origin/destination city, state, ZIP (with map picker), pickup and delivery windows, equipment type, weight, dimensions, hazmat toggle, rate type (flat rate or "make offer"), optional rate amount, notes, and contact preference.
   - Attestation statement displayed prominently above the submit button.
   - Auto-expire loads after 7–14 days; status transitions to filled or expired from dashboard.
   - Support file attachments (e.g., COI) stored in S3-compatible storage.
3. **Carrier Discovery & Engagement**
   - Home page search with quick filters.
   - Results page with server-rendered list sorted by newest first; each card highlights origin, destination, dates, equipment, rate info, and shipper verification badge.
   - Advanced filters: origin radius, destination radius, pickup window, equipment type, minimum rate, maximum weight, and full-text search on notes/company.
   - Save search criteria for quick access from dashboard; email alerts in later iterations.
   - Load detail page with full description and contact options.
4. **Messaging & Contact**
   - In-app chat between shipper and carrier tied to a load; notifications via email.
   - Masked email relay that forwards messages while hiding real addresses until both parties opt to reveal phone numbers.
   - Phone numbers remain hidden until mutual opt-in.

## Trust & Safety
- Display verification badges (email, phone, DOT/MC, business domain).
- After a completed match, allow simple 1–5 star rating with comment (future iteration).
- Basic moderation tools: flagging loads or users, quick takedown workflow, and audit log of admin actions.
- Rate-limit signups and postings by IP and email domain; block disposable email providers.
- Prominent disclaimer: "We publish listings only—no brokering, escrow, or rate confirmations."

## Data Model (v1)
- **Users**: role, email verification flag, phone, password hash, timestamps, plus DOT/MC or business credentials depending on role.
- **Loads**: shipper owner, origin/destination details with geocodes, pickup/delivery windows, equipment, weight, dimensions, hazmat flag, rate type & amount, notes, status, created/expiration timestamps.
- **Messages**: load reference, sender, recipient, body, timestamps.
- **Saved Searches**: user reference, search parameters JSON, created/updated timestamps.
- **Flags**: target type/id, reason, reporter, created timestamp, resolution metadata.
- **Ratings (future)**: as outlined above.

## Pages & UX
- **Home/Search** – Craigslist-style list with simple search bar and quick filters.
- **Results** – Paginated list with filters sidebar (minimal JS for radius sliders and autocomplete).
- **Post Load** – Verification status banner, form with required fields, non-broker attestation checkbox.
- **Load Detail** – Full load info, verification indicators, messaging CTA.
- **Auth (Login/Signup)** – Simple forms with email OTP verification.
- **Dashboard** – Tabs for My Loads, Saved Searches, Messages, Account settings.
- Responsive layout using basic HTML and lightweight CSS (utility classes). Server-rendered templates with progressive enhancement only for search filters and map pickers.

## Technical Stack
- **Backend**: Django or Rails monolith with server-rendered HTML templates.
- **Database**: PostgreSQL using built-in full-text search for notes/title.
- **Cache**: Redis for sessions, rate limiting, and caching hot search queries.
- **Storage**: S3-compatible bucket for attachments (COI, documents).
- **Auth**: Native auth with email OTP, optional phone OTP via Twilio later.
- **Verification Integrations**: DOT/MC public lookup APIs; business email domain validation.
- **Hosting & Ops**: Deploy on Fly.io/Render/Heroku, fronted by Cloudflare for DNS, caching, and basic WAF. Instrument with Sentry, logs, and health checks.

## Search & Filtering
- PostgreSQL `to_tsvector` on load notes, shipper company, and route metadata.
- Geospatial filtering via Haversine formula on stored lat/long for origin/destination.
- Sorting by newest first; add "best match" scoring in later releases.

## Moderation Workflow
- Users can flag loads or profiles; flagged items enter admin queue.
- Admin dashboard shows reason, reporter, history; supports quick takedown (status toggle) and creates audit log entries.
- Manual review for early-stage operations; automation rules added later for repeat offenders or suspicious patterns.

## Compliance & Legal Notes
- Terms of Service explicitly prohibit broker postings, double-brokering, and mandate truthful listings.
- Privacy policy clarifies contact info visibility and data retention.
- Platform messaging consistently states it is a listing board only—no dispatching, rate confirmation generation, or payment handling.

## Future Enhancements
- Phone OTP verification, push/email alerts for saved searches, analytics dashboard for shippers, integrations with ELD/telematics for automated status updates, and machine-learning-based fraud detection.


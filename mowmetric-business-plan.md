# MowMetric — Business Plan

*An open marketplace for lawn care leads. Verified at submission, priced by quality, decaying by age, re-listable on consumer demand.*

Last updated: April 19, 2026

---

## 1. The thesis in one paragraph

Homeowners can't figure out what lawn care should cost, so they either overpay or never hire. Providers can't find customers without paying extortionate rates to HomeAdvisor-style push networks that share the same lead with 4–6 competitors and still charge $60–$100 per lead. MowMetric fixes both sides with a single mechanism: a satellite-measured, BLS-anchored price calculator that gives the homeowner a real number, captures consent, verifies their contact details, scores the lead, and drops it into an open self-serve marketplace where any provider can buy it exclusively. Unsold leads decay in price over time. Leads where verification fails are monetized as a separate "Field Visit" tier for door-knocking providers. Consumers who aren't satisfied with their first provider can re-list themselves with one click from their email — for the full 60-day life of the lead — to get additional competitive quotes.

---

## 2. Why the existing model is broken

The incumbent lead-gen playbook — HomeAdvisor, Angi, Thumbtack, Porch, 99Calls, LawnStarter's enterprise arm — is a push-shared model. A homeowner fills out a form, the platform sells the lead simultaneously to 3–6 providers at $30–$100 each, and whoever calls first wins. The provider close rate on a shared lead is typically 8–15%, so effective CAC lands at $250–$800. HomeAdvisor/Angi paid a $7.2M FTC settlement in 2023 for the exact abuses this model produces: dead leads, leads sold without consent, charges for leads that never wanted the service. The most common provider complaint across every incumbent is the same: "half the contacts don't pick up and half the numbers are disconnected."

Adjacent industries already solved this. Insurance lead-gen (EverQuote, QuoteWizard) caps shared leads at 3 buyers and offers self-serve ZIP-filtered dashboards. Real estate (BoldLeads, RedX, Zillow Premier Agent) sells aged leads at tiered discounts and lets agents subscribe to territories. Home services has not caught up.

MowMetric's opportunity is to port the insurance/real estate marketplace structure into lawn care, with verified contact details as a core differentiator, while the competition is still selling unverified 2015-era lead lists.

---

## 3. The product

### 3.1 Consumer side

A calculator that takes an address, measures the lawn via satellite, applies BLS labor data + local market rates, and returns an honest price range for mowing, fertilization, aeration, and landscape maintenance. The homeowner can:

- Take the estimate and walk away (no provider contact required).
- Opt in to matching by providing phone, email, and consent.
- Re-list themselves at any point in the lead's 60-day life to get additional competitive quotes, via a one-click button in their email confirmation.

**Consent language** at form submission covers the full scope of contact modes:

> "By submitting this form, I consent to being contacted by up to 3 local lawn care providers via phone, email, or in-person visit at the address provided. I may request additional quotes or opt out at any time."

**Privacy commitments** are load-bearing: no data brokers, no hidden resale, explicit opt-out at submission, max 60 days in system, delete-on-request within 30 days.

### 3.2 Provider side (the marketplace)

A map-centric dashboard showing every available lead, filtered by ZIP, score, service type, and verification state. Every lead appears as a card with:

- ZIP, city, approximate lot size
- Quality score (0–100) with visible factors
- **Verification badges** — Email verified / Phone verified / Local area code / Address confirmed
- Service type and intent signal (one-time vs. seasonal)
- Current price (with the next scheduled markdown displayed inline — "drops to $41 in 14h")
- Age in hours/days
- Any special flags: **"Hot re-list — consumer actively shopping"**, **"One-shot lead — no re-list possible"**, or **"Field Visit only"**

Providers pay via Stripe Checkout. On purchase, the real phone number and real email are unlocked immediately in the portal and delivered via transactional email to the provider. No proxies, no masking, no middleware. The provider owns the contact and the relationship from that moment forward.

### 3.3 Subscription tiers

- **Free Watcher** — browse the marketplace, buy anything
- **$29/mo Responder** — email alerts for up to 5 saved ZIPs, real-time when leads appear
- **$79/mo Priority** — unlimited ZIPs, SMS + email alerts, advanced filters (size, intent, grass condition), **10-minute head start** before leads become public
- **$49/mo Aged Archive** — unlimited browsing of 30–60 day old leads at flat $3–$5 pricing

The Priority head start is the subscription's real value. It doesn't lock competitors out of a ZIP (which is what kills the monopoly dynamic that sinks territory-exclusive models); it just gives paying subscribers a short window to grab the cream before everyone sees it.

### 3.4 Provider signup — minimal by design

Signup is intentionally thin to remove friction for legitimate providers:

- Business name (free text)
- Email + password
- Phone number with SMS verification
- Stripe payment method
- Click through the Terms of Service

That's the entire flow — a provider can be signed up and buying a lead in under three minutes. No business license upload, no insurance certificate, no DUNS number, no account volume throttling. Stripe's own fraud detection filters the payment-side bad actors; SMS verification blocks trivially fake signups.

**Optional "Verified Business" badge** — providers can upload a license, insurance certificate, or both to earn a Verified badge on their profile. Purely opt-in, never a gate to buying leads, but a visible trust signal to subscription-tier providers and a likely conversion driver over time.

---

## 4. Lead scoring, contact verification, and the Field Visit tier

A lead's score is the single most important number on the platform — it drives price, subscriber alerts, and provider trust. The score has two halves: **job signal** (what kind of work the lead represents) and **contact quality** (whether the human on the other end is real and reachable). Each half tops out around 50 points for a combined 100.

### 4.1 Job signal — ~50 points

Computed from the form + the satellite measurement. Key components:

- Lot size and complexity (larger, higher-revenue jobs score higher)
- Service type and urgency (recurring mow season commitments > one-off cleanups)
- Local demand density in the ZIP (more competing providers → more liquid, scored higher)
- Grass condition signals self-reported on the form
- Timeline ("this week" >> "sometime this summer")

### 4.2 Contact quality — ~50 points, verified twice

Every lead is validated synchronously at form submission, before the thank-you page, adding roughly 400ms of latency. A **second verification pass runs at sale time**, right before the provider's contact info is unlocked, to catch the case where a phone was verified 10 days ago at submission but has since been disconnected. If sale-time verification fails, the purchase is blocked before the provider's card is charged — they never pay for a dead contact.

**Email validation (NeverBounce / ZeroBounce / Kickbox, ~$0.006 per check):** syntax + MX + SMTP deliverability. Returns valid / invalid / accept-all / disposable / role-based / unknown.

| Email status | Score contribution |
|---|---|
| Valid + personal domain (gmail, yahoo, icloud, outlook, ISP) | +15 |
| Valid + custom personal domain | +10 |
| Accept-all (unconfirmable but deliverable) | +3 |
| Disposable, role-based, or invalid | Scored as failed email |

**Phone validation (Twilio Lookup, ~$0.005 per check):** E.164 format + carrier + line type + in-service status.

| Phone status | Score contribution |
|---|---|
| Mobile, carrier confirmed, area code matches submission region | +15 |
| Mobile, carrier confirmed, non-local area code | +10 |
| Landline, in-service | +8 |
| VoIP (Google Voice, TextNow, etc.) | +3 |
| Disconnected / invalid / unassigned | Scored as failed phone |

**Cross-checks (all free or near-free):**

- Phone area code region matches billing address state: **+5**
- Email name component matches a name on the form: **+5**
- Three+ leads from same IP in 24h with different names: auto-flag for review

**Behavioral signals (free):**

- Hidden honeypot field: submissions that fill it are auto-discarded
- Time-to-submit under 8 seconds: bot-like, penalized
- reCAPTCHA v3 score: passive feed into the score

### 4.3 Verification-state routing — what happens when one or both checks fail

This is the differentiated structure. A failed verification doesn't kill the lead; it re-routes it.

| Verification state | Score adjustment | Routing | Notes |
|---|---|---|---|
| Both verified | 0 | Main Marketplace at full tier price | Normal behavior |
| Email ✓, phone ✗ | **−20 points** | Main Marketplace, lower tier | Close rate hit; email still works, so re-list is still possible |
| Phone ✓, email ✗ | **−15 points** | Main Marketplace, lower tier, **flagged "One-shot lead — no re-list possible"** | Phone works for provider, but MowMetric can't re-engage consumer (no email), so re-list button is disabled |
| Both failed | Moved out of Main Marketplace | **Field Visit tier** | Flat $3 (or $5 for large lots). Address product only — no phone/email delivered to provider. |

**Why phone failure is penalized more heavily than email failure:** lawn care sales happen on the phone. Writing an email to request a callback introduces drop-off at every step, and close rates run 30–40% lower than a phone-verified lead.

**Why email failure still matters despite the smaller score hit:** it breaks MowMetric's entire re-engagement loop. No confirmation email. No "Get more quotes" button reaching the consumer. The lead becomes a one-shot sale with no second-sale revenue potential. The score deduction is smaller, but the lead's flagged on the provider card so buyers understand they're buying a one-and-done.

### 4.4 The Field Visit tier

Leads where both phone and email fail verification still carry value. The address is real, the lot measurement is real, the person filled out a form asking about lawn care — the remote-contact channels just aren't usable. Door-knocking providers (a real segment — newer crews without ad budgets, college summer crews, neighborhood-focused operators) can convert these in person.

Field Visit leads appear in a separate section of the marketplace, clearly labeled, at flat pricing:

- **$3 flat** for standard lots
- **$5 flat** for large lots (potentially high-value jobs)

On purchase, the provider gets:

- Full address
- Lot size and satellite measurement
- Form-reported service type, grass condition, timeline
- Score (computed on job signal alone)
- A clear note: **"Contact verification failed. Recommended for in-person outreach only. Phone and email are not provided."**

The unverified phone and email are **never** handed over. Delivering numbers that don't work is a TCPA-risk creator and a provider-trust destroyer. Field Visit leads are an address product, not a contact product.

**Field Visit consent** is already covered by the submission language that includes "in-person visit at the address provided." Consumers who aren't comfortable with door visits shouldn't submit the form — the calculator estimate is available without submitting.

### 4.5 Front-end recovery — don't let typos dump leads into lower tiers

Three cheap client-side additions recover 3–5% of submissions that would otherwise fall into a degraded tier:

- Email typo detection with suggestion ("did you mean gmail.com?") for common misses (`gmial`, `yaho`, `hotnail`, etc.)
- Phone format validation that rejects too-short/too-long numbers before submission
- A post-submission thank-you page with a "check your email — if you don't see it in 5 minutes, update your email here" one-field correction form

Small engineering lift, meaningful tier-distribution improvement.

### 4.6 Why verification is strategic, not hygienic

Verified contact quality is the most powerful word-of-mouth hook among contractors. Lawn care providers talk to each other on Facebook and in local trade groups. "Their numbers actually pick up" is the kind of thing that travels. Every incumbent treats verification as an afterthought; MowMetric treats it as product-one.

---

## 5. Pricing, decay, and consumer re-listing

### 5.1 What a lawn care provider can actually afford

Industry benchmarks validated against operator P&Ls and trade publications (Lawn & Landscape, Turf Magazine, NALP data):

- Average annual revenue per residential customer: **$1,875**
- Net margin on residential route work: **15–25%**, midpoint 20%
- Average customer retention: **3 years**
- Net lifetime value: **~$1,125** (conservative) to **~$2,812** (optimistic bundled)
- Target CAC:LTV ratio: **3:1 or better**, so max defensible CAC: **$375–$937**
- Typical exclusive-lead close rate: **40–55%** on MowMetric (leads are pre-verified)

A provider paying $55 for a lead that closes at 45% has effective CAC of $122 and LTV:CAC of roughly 9:1. Excellent for the provider *and* healthy top-of-funnel revenue for MowMetric.

### 5.2 Lead price tiers (at listing, before decay)

| Score | Price | Est. close rate | Provider CAC | Provider LTV:CAC |
|---|---|---|---|---|
| 85–100 | $55 | 45% | $122 | 9:1 |
| 65–84 | $32 | 35% | $91 | 12:1 |
| 45–64 | $18 | 22% | $82 | 14:1 |
| 25–44 | $8 | 12% | $67 | 19:1 |
| 0–24 | $3 | 5% | $60 | 19:1 |
| Field Visit | $3–$5 flat | 5–15% (door-knock) | $30–$100 | 11:1–37:1 |

Every tier leaves the provider with CAC well inside the $375 ceiling.

### 5.3 Price decay — automated markdown by age

A nightly cron job updates `current_price` on each lead based on age:

| Age | Multiplier | $55 lead becomes | $32 lead becomes |
|---|---|---|---|
| 0–24 hrs | 1.00× | $55 | $32 |
| 24–72 hrs | 0.75× | $41 | $24 |
| 3–7 days | 0.50× | $28 | $16 |
| 7–14 days | 0.30× | $17 | $10 |
| 14–30 days | 0.15× | $8 | $5 |
| 30–60 days | Aged archive flat | $3 | $3 |
| 60+ days | Deleted | — | — |

Rules:

- The next scheduled markdown is shown on the card: "drops to $41 in 14h." Creates urgency on fresh leads and deal-of-the-day pull on older ones.
- Priority subscribers can set "auto-buy if price drops below $X" on saved ZIPs — the quiet monetization of the aged pool.
- Field Visit leads do not decay (they're already at rock-bottom pricing) but still retire at 60 days.

### 5.4 Sales model — one buyer at a time, no holds, no hand-holding

Every lead is sold exclusively to one buyer at a time. On purchase:

- The provider's card is charged via Stripe Checkout.
- The real phone number and real email are unlocked instantly in the provider portal and emailed to the provider.
- The lead is removed from the marketplace.
- The provider owns the contact from that moment on.

**No 24-hour contact windows. No "confirm I contacted them" buttons. No 14-day exclusive holds. No proxy routing. No call or message logging.** The provider paid for the lead — they own it, and they use it how they want.

**No refund mechanism.** Every lead is buy-at-your-own-risk. The only failure mode we protect against is a phone/email that's already dead at the moment of sale, which the second verification pass catches before the purchase completes. Once a provider has the real contact info, the outcome is theirs to own.

### 5.5 Consumer-initiated re-listing

The form promises "match me with up to 3 providers." If only one ever reaches out, or if the first provider is a bad fit, overquotes, or never calls, the consumer got less than they signed up for. This is solved by a consumer-controlled re-list button rather than automatic time-based re-sale.

**How it works:**

- The consumer receives a confirmation email within 2 hours of their lead being sold to the first provider: *"A local pro has your estimate and will reach out shortly. If their quote doesn't feel right, or you'd like more options, you can request additional competitive quotes any time — [Get More Quotes]."*
- The button works for the **full 60-day life of the lead**. No arbitrary 14-day cutoff.
- Clicking the button re-lists the lead on the marketplace at the current decay-tier price with a **"Hot re-list — consumer actively shopping"** badge.
- Consumer can click the button up to 2 more times (3 providers total, matching the consent ceiling).
- The lead still auto-deletes at 60 days per the privacy promise. After that, if the consumer still wants quotes, they resubmit the form — which is effectively one-click for returning visitors (address cookied, pre-filled).

**Why this is better than automatic sequential re-sale:**

- Re-listed leads carry the strongest possible signal: *this consumer is actively shopping right now and unhappy with their current options.* Close rates on Hot re-lists probably exceed 45% even at 1-day age.
- Consumers aren't ambushed by unsolicited second and third calls the day after they've already booked.
- Platform stays out of the business of deciding when a consumer "needs" more quotes — the consumer decides.
- Every re-list is a re-engagement touch, turning MowMetric into a relationship rather than a one-time transaction.

**Restrictions:**

- Re-list button is **disabled for leads flagged "One-shot" (email verification failed)** — there's no way to deliver the button if email doesn't work. These leads sell once, full stop.
- Re-list button is **disabled for Field Visit leads** — the consumer never received a MowMetric confirmation email to click a button in, because email verification failed.

### 5.6 Exclusive-per-lead, not exclusive-per-territory

This entire pricing structure sells exclusivity on a per-lead basis, not by ZIP code or region. Any provider can buy any lead in the marketplace. No single operator can monopolize a ZIP by purchasing a territory subscription — the subscriptions MowMetric sells grant visibility and alert speed, not buying exclusivity.

---

## 6. Operations — how this runs with near-zero manual work

The automation is the business. If you're manually dispatching leads to providers, you're a staffing agency with a landing page. If the marketplace runs itself, you're a software company with software margins.

### 6.1 Tech stack

- **Frontend**: Next.js (existing calculator extended into portal)
- **Backend**: Node/TypeScript API on Vercel or Railway; Postgres (Supabase is fine)
- **Payments**: Stripe Checkout + Stripe Billing for subscriptions
- **Email deliverability validation**: NeverBounce, ZeroBounce, or Kickbox
- **Phone validation**: Twilio Lookup API
- **Transactional email**: Postmark or Resend for confirmations, alerts, and re-list button delivery
- **SMS**: Twilio for Priority alerts and optional consumer match notifications
- **Lawn measurement**: existing satellite/Mapbox pipeline
- **Cron**: Vercel Cron or Upstash QStash for nightly decay job + 60-day purge

### 6.2 Fully automated flows

1. **Consumer submits form** → email + phone validated synchronously (~400ms) → lead scored → consent timestamped → address verified via Mapbox reverse geocode. Routed to Main Marketplace, Field Visit tier, or dropped based on verification state.
2. **Priority subscribers** matched to lead by ZIP + filters → email/SMS fired with 10-minute head-start banner.
3. **Lead goes public** to the marketplace at T+10 min, with verification badges and any special flags rendered on the card.
4. **Provider clicks Buy** → sale-time re-verification runs on phone and email → if fresh, Stripe Checkout proceeds → webhook unlocks contact info in portal and emails it to provider. If sale-time verification fails, purchase is blocked and the lead is demoted or retired.
5. **Consumer receives confirmation email 2 hours post-sale** with the re-list button (only if email verified).
6. **Consumer clicks re-list button** → lead re-enters marketplace with "Hot re-list" badge at current decay-tier price.
7. **Nightly cron** updates prices per decay table, moves leads past 30 days into aged archive, deletes leads past 60 days.
8. **Weekly digest email** to Watchers showing top leads in their region (conversion funnel into paid tiers).

The founder's only operational jobs during normal operation are customer support tickets, abuse review, and growth.

### 6.3 Dashboards you still want to build (for yourself)

- Lead lifecycle funnel: created → validated → listed → sold → re-listed → re-sold
- Verification pass/fail rates by source (SEO, paid, referral) — flags junk traffic sources early
- Field Visit tier sell-through rate
- Cohort revenue per lead (to tune the decay curve)
- Provider repeat-purchase rate (leading indicator of marketplace health)
- Provider complaint rate per lead (correlates with false positives in the score)

---

## 7. Solving the chicken-and-egg — sequenced launch

The marketplace must look populated on day one of provider signups, or providers bounce and never come back. The fix: delay provider signups behind the consumer launch.

### Phase 1 (Months 1–2): Calculator-only mode

- Public calculator launches with no provider marketplace.
- Contact verification is live from day one — this lets you throw away garbage traffic and confirm the validation pipeline works before providers ever see it.
- Consumers get their estimate. Optional CTA: "Get 3 quotes from local pros — we'll notify you when we open in your area."
- Match consent captured with explicit timestamp + language.
- SEO, content, and local ads drive traffic. Goal: **50–200 scored leads accumulated** plus a populated Field Visit tier, and a handful of geographic hot spots.

### Phase 2 (Month 3): Provider waitlist + beta onboarding

- Cold outreach to 200–500 local LCOs via Facebook groups, Google Maps scraping, NALP directory. Pitch: "We have N real leads waiting in your ZIP — every contact verified."
- Launch a provider waitlist page with ZIP-level lead counts ("We have 7 leads in 37209 right now, plus 3 Field Visit opportunities").
- Beta onboard 20–50 providers, free access to the first 30 days, $0 subscription, pay only for leads purchased.

### Phase 3 (Month 4+): Full launch

- Marketplace opens to anyone who completes the minimal signup flow.
- Aged archive and Field Visit tier are already populated — new providers can start with cheap test purchases instead of gambling on a $55 top-tier lead before they trust the platform.
- Subscriptions turn on.

Aged archive disclosure is non-negotiable: these leads are 30+ days old, close rates are 2–5%, price reflects that. Never market them as "hot."

---

## 8. Unit economics — MowMetric's P&L per lead

### Per-lead expected revenue (over its 60-day life, including potential re-lists)

Blended expected revenue per 85+ lead, factoring decay, sell-through, and consumer re-list probability:

- Initial sale expected revenue: ~$40 (full price weighted against decay likelihood)
- First re-list: 35% probability × ~$30 = ~$10.50
- Second re-list: 10% probability × ~$18 = ~$1.80
- **Expected revenue per 85+ lead: ~$52**

Rough expected revenue per mid-tier lead (45–64 score): **~$15**
Rough expected revenue per low-tier lead (0–44 score): **~$4**
Rough expected revenue per Field Visit lead: **~$1.50** (50% sell-through at $3 flat)

### Per-lead marginal cost

- Mapbox geocode + satellite measurement: ~$0.02
- Email deliverability validation at submission: ~$0.006
- Phone validation at submission: ~$0.005
- Sale-time re-verification (runs only on sold leads, amortized): ~$0.009
- Transactional email (confirmations, re-list button, delivery): ~$0.03
- SMS (on Priority alerts only, amortized): ~$0.02
- Stripe fee on avg $25 lead sale: ~$1.00
- Compute/storage amortized: ~$0.03
- **Total marginal cost per sold lead: ~$1.12**

Gross margin per high-tier sold lead: **~97%.** This is software margin. Verification is noise against the per-lead revenue.

---

## 9. Year 1 projection — honest case

Three scenarios. All assume the Phase 1 → Phase 3 sequenced launch, one founder + occasional contractor support, and $15–25K in Year 1 spend on ads, SEO content, validation services, and tooling.

### 9.1 Assumptions

- **Submission mix after verification routing**: ~85% to Main Marketplace (split across tiers), ~15% to Field Visit tier
- **Main Marketplace score distribution**: 20% 85+, 30% 65–84, 25% 45–64, 15% 25–44, 10% 0–24
- **Blended expected revenue per submission**: ~$18
- **Subscription attach rate at maturity**: 15% of active providers on $29/mo, 5% on $79/mo, 3% on $49/mo aged

### 9.2 Conservative — "we hit friction everywhere"

- Month 12 active monthly submission volume: **300/mo**
- Active providers: **60**, subscription revenue: **~$450/mo**
- Year 1 total lead revenue: ~$26K
- Year 1 total subscription revenue: ~$2.5K
- **Year 1 total revenue: ~$28K**

### 9.3 Base — "it works, slowly"

- Month 12 active monthly submission volume: **900/mo**
- Active providers: **180**, subscription revenue: **~$1,800/mo**
- Year 1 total lead revenue: ~$82K
- Year 1 total subscription revenue: ~$9K
- **Year 1 total revenue: ~$91K**

### 9.4 Optimistic — "SEO compounds + a regional wedge works"

- Month 12 active monthly submission volume: **2,500/mo**
- Active providers: **400**, subscription revenue: **~$5,200/mo**
- Year 1 total lead revenue: ~$225K
- Year 1 total subscription revenue: ~$28K
- **Year 1 total revenue: ~$253K**

### 9.5 Likelihood assessment

Base case is the one to plan around. Conservative is what happens if the sequenced launch flops in one or two geographies; you'd stay in that range while you re-do GTM. Optimistic requires one of three wedges working — a viral moment, a strong local-SEO footprint in 2–3 metros, or a BD partnership with a franchise group that pushes providers onto the platform.

The thing that actually determines which scenario you hit is not the product — the product is the easy part — it's whether you can fill the marketplace with consumer leads cheaply. That's an SEO and content-velocity problem. "How much does lawn mowing cost in [city]?" is a high-intent long-tail that's under-served nationally. If you ship 200 city pages with real local pricing in the first 90 days, base case becomes likely. If you ship 20, conservative is likely.

Verified contact + the Field Visit tier + consumer-initiated re-listing each make the base and optimistic cases more defensible: every lead sold actually reaches a human (or is clearly labeled for door-visit only), close rates run toward the high end of the benchmark, leads get two or three bites at the revenue apple, and provider word-of-mouth becomes the cheapest acquisition channel for the second side of the market.

---

## 10. Risks and what kills this

- **Liquidity failure**: you open the marketplace and there aren't enough high-quality leads. Mitigated by the sequenced launch, but fragile. Counter: focus launch on 2–3 geographies, go deep, then expand.
- **TCPA exposure**: the 11th Circuit struck down the FCC's one-to-one consent rule in January 2025, which is helpful, but state-level risk (Washington, Florida) is still real. Counter: explicit consent at submission with timestamp, limit provider contact attempts, honor opt-outs automatically, never hand out unverified phone numbers, keep lead age ≤60 days.
- **Provider misuse / lead resale**: someone buys a lead and tries to resell the contact info. Counter: strong Terms of Service prohibiting resale, consumer-side reporting mechanism ("If you get contacted by anyone other than [Provider], reply to this email"), terminated providers forfeit account credit balance. No heavier infrastructure until abuse becomes real — this is a regional lawn-care marketplace, not a national data brokerage, and the resale economics for geographic lawn-care leads are poor.
- **Verification false positives**: legitimate leads get routed to Field Visit or lower tiers because someone uses a long-held VoIP line. Counter: never fail a lead entirely on a single signal; monitor pass rates by traffic source and re-calibrate thresholds.
- **HomeAdvisor / Angi response**: they won't pivot to this model (it cannibalizes their economics), but they could undercut pricing regionally. Counter: their brand is now negative with providers; price war actually helps you.
- **SEO plateau**: you rank page 2 for the bigger terms. Counter: hyper-local long-tail content + earned links via an "average cost in your city" data study published quarterly.

---

## 11. What to build first, in order

1. Marketplace schema + provider auth + Stripe Checkout
2. **Contact verification pipeline** (NeverBounce + Twilio Lookup integration at form submit *and* at sale)
3. Lead scoring algorithm with job-signal + contact-quality halves, including verification-state routing
4. **Field Visit tier** — separate marketplace section, flat pricing, address-only product
5. Provider portal with map view, lead cards, and verification badges
6. Transactional email + SMS plumbing (Postmark + Twilio)
7. **Consumer re-list button** — confirmation email + 60-day re-list flow with Hot re-list badging
8. Nightly decay cron + 60-day purge
9. Priority head-start queue (10-min delay gate)
10. Aged archive subscription gate
11. Admin dashboards (lifecycle funnel, verification pass rates, Field Visit sell-through)
12. Weekly digest email to Free Watchers
13. Front-end typo / format recovery UX

Realistic build timeline for a focused founder + occasional contract help: **6–10 weeks** to a marketplace that can onboard real providers. Verification is 2–3 days of incremental work. The re-list button is maybe 2 days.

---

## 12. Bottom line

The mechanism works on paper and survives pressure-testing against real provider P&L. The automation requirements are not exotic — everything here is a 2026-standard SaaS build. The model stays disciplined: verify at submission and at sale, sell exclusively to one buyer per slot, let consumers re-list themselves when they want more options, monetize the leftover contact-failed leads as a door-knock product, and stay entirely out of the communications-middleware business.

Contact verification + the Field Visit tier together eliminate the two worst failure modes of every incumbent — bad contact info and wasted potential leads — while keeping the operational surface area small. The consumer re-list button keeps the platform respectful of consumers and responsive to their real shopping behavior without auto-spamming them with follow-up outreach.

The likelihood of hitting the base case (~$91K Year 1 revenue, 180 active providers) is reasonable if the sequenced launch executes and the content velocity hits 150+ city pages in the first six months. The thing most likely to sink it is not the model or the tech; it's whether the founder can keep consumer lead flow ahead of provider demand during the first 90 days after the marketplace opens.

The upside case — where SEO compounds and one metro becomes a wedge — is what makes this a business worth building, not just a consulting product.

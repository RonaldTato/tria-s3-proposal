# Tria Ambassador Calculator — Team Explainer

> Plain-English explainer for the team. What we built, why we built it this way, and what's intentionally left out. Pair with `calculator-spec.md` if you want the implementation details.
>
> Live: https://ronaldtato.github.io/tria-s3-proposal/landing/#calculator

---

## TL;DR

A public-facing ambassador earnings calculator. One slider (active referrals), one big number (12-month projected rewards under the Average Pusher band), plus an "upper-bound" High Pusher band so the visitor sees what a mature, high-volume network earns. Inspired by ether.fi's affiliate calculator UX but with Tria's actual tier rates and a two-band projection layer added on top — no "conservative" projection because the audience for this page is operators who already move volume.

---

## What it does

The visitor moves a slider from 0 to 500 active referrals. The calculator auto-detects which tier they're in (Bronze 1 through Gold based on the Season 3 spec), then projects their annual rewards two ways:

- **Average Pusher** — a crypto-native network where most referrals actually trade and use the card as a primary payment method. This is the realistic floor for a network that's actually moving.
- **High Pusher** — a serious trader desk or KOL community where Premium cardholders, real-volume traders, and mature engagement are the norm. Same network at mature operating velocity.

The headline number on the page is the **Average Pusher** band's 12-month total. **High Pusher** sits next to it as the upper bound. Below both, a breakdown shows where the Average Pusher number comes from (futures fees vs. card spend vs. card sales) with each stream's % share of the total.

**There is no "Conservative" projection on purpose.** The audience for this page is operators who already move volume — KOLs, MLM veterans, trading community leaders. A "casual user" floor would undersell the program and feel disconnected from how this audience actually operates.

---

## Why we built it this way

The earlier version of the calculator had eight input fields: active refs, monthly trading volume per ref, monthly card spend per ref, Virtual cards sold/month, Signature cards sold/month, Premium cards sold/month, sub-ambassadors, plus a "card sale value" slider before that. Too dense for a public surface — a visitor lands on it and freezes.

**Three things forced the redesign:**

1. **KOLs don't know most of those inputs.** "Average trading volume per referral" isn't a number anyone has handy. We have to bake it into reasonable defaults and let them adjust the granularity through a band, not a knob.
2. **Card product detail isn't public-surface material.** The ambassador portal has the Virtual / Signature / Premium breakdown. The landing page shouldn't.
3. **Comparison reference.** ether.fi's calculator does exactly the right thing for its audience: one slider, one big number, three projection trajectories. We borrowed the one-slider structure but built it differently (two bands shown side-by-side instead of a chart, baked Tria rates, and our own per-band assumptions — and we dropped their "Conservative" because our audience isn't conservative).

---

## How to read the calculator

1. **The slider sets active referrals.** Per the Season 3 spec, that's anyone in your network with ≥$1 card spend or ≥$10K trading volume in the last 90 days, counted at depth 1 + depth 2.
2. **Tier badge updates as you slide.** Thresholds: Bronze 1 (0–2) · Bronze 2 (3–9) · Silver 1 (10–19) · Silver 2 (20–29) · Gold (30+).
3. **Headline number = Average Pusher band annual.** USD-equivalent, claimable in USDC/USDT in-app.
4. **Two band cards.** Average Pusher (featured) · High Pusher — each shows annual + approximate monthly.
5. **Breakdown panel.** Shows the three streams (futures / card spend / card sales) for the Average Pusher band, with each stream's % share so you can see where the money comes from at any scale.
6. **Higher hierarchy callout.** A single line acknowledging there's a higher tier above Gold but no specifics.
7. **Three collapsible FAQs** at the bottom: band assumptions table, indirect commission explainer, rate application notes.

---

## The two bands — what each assumes

Each band's only difference is the **per-referral activity profile**. The math is identical across bands; only these three inputs change:

| Per referral | Average Pusher | High Pusher |
|---|---:|---:|
| Monthly trading volume | $500K | $2M |
| Monthly card spend | $2,000 | $5,000 |
| Card-subscription sales / year | $200 | $1,000 |

**Average Pusher** ≈ a crypto-native community where most referrals actually trade on Tria and use the card as their primary payment method. This is what we'd expect a real KOL network to look like 6 months in — not a "casual user" floor.

**High Pusher** ≈ a serious trader desk or KOL community with mature engagement — power users, Premium cardholders, real monthly trading volume. Roughly what the same network looks like a year in with product fit working.

The "card-subscription sales" line is the gross dollar value of cards your network buys per year, including new issuance, upgrades, and replacements. It absorbs the product mix (Virtual $20 / Signature $90 / Premium $225) at a middle-ground avg price of ~$100/card. **Calibrated against ambassador throughput at a 30-ref Gold network:** Average Pusher = ~5 cards/month sold; High Pusher = ~25 cards/month.

This way the visitor never sees Virtual / Signature / Premium in the UI. The product detail stays in the portal.

**Why no "Conservative" band:** the audience for this page (KOLs, MLM veterans, trading community leaders) does not operate at the casual end. A casual-user projection would make the headline number small, which would feel disconnected from how this audience reads themselves. Both bands assume the network is moving; the only question is at what velocity.

---

## Worked examples

### 30 active referrals (Gold tier)

| Band | Monthly | Annual |
|---|---:|---:|
| **Average Pusher** | **~$1,425** | **$17,100** |
| High Pusher | ~$5,625 | $67,500 |

### 100 active referrals (Gold)

| Band | Annual |
|---|---:|
| **Average Pusher** | **$57,000** |
| High Pusher | $225,000 |

### 500 active referrals (Gold) — KOL scale

| Band | Annual |
|---|---:|
| **Average Pusher** | **$285,000** |
| High Pusher | **$1,125,000** |

A KOL with 500 active referrals running a real trading community is modeling **a quarter million a year** on the Average Pusher band — and **just past a million** on High Pusher — from direct commission alone, on the public-ladder Gold rate. That's the headline that has to land.

---

## What's deliberately left out

- **Indirect commission.** Sponsoring sub-ambassadors compounds the math significantly (locked rates 10% / 0.05% / 5% on their direct customers' transactions — **one level only**, not infinite cascading). We mention it in the FAQ and give a rough heuristic — 5 sponsored ambassadors ≈ 2× annual run-rate — but it's not in the slider math. Decision: keep the headline focused on direct activity so the visitor doesn't conflate "what I can build" with "what my sponsors can build for me." Higher hierarchy operators (Platinum, Diamond) earn deeper revenue via a separate "spread to max tier" mechanism, but the standard program caps indirect at one level.
- **Higher hierarchy.** The calculator caps at Gold. There's a one-line callout acknowledging a higher hierarchy exists, accessible by accelerated climb or direct invitation. No rates, no tier names, no specifics. (This was explicit direction from the previous iteration.)
- **Card product specifics.** Virtual / Signature / Premium and their respective prices never appear on the public surface. Baked into the per-band card-activity assumption.
- **Competitor comparisons.** No ether.fi side-by-side or similar. The page focuses on Tria's own economics.

---

## What's tunable (in case we need to adjust)

These are the levers that determine the calculator's output. Adjusting any of them is a single constant change:

| Lever | Current value | Where set |
|---|---|---|
| Tier thresholds | 3 / 10 / 20 / 30 active refs | TIERS array |
| Tier rates | per spec lines 60–66 | TIERS array |
| Tria futures fee | 0.05% of volume | `TRIA_FUTURES_FEE` |
| Average Pusher volume | $500K/ref/mo | `BANDS.averagePusher.volume` |
| Average Pusher card spend | $2K/ref/mo | `BANDS.averagePusher.spend` |
| Average Pusher card sales | $200/ref/yr | `BANDS.averagePusher.cardActivityAnnual` |
| High Pusher volume | $2M/ref/mo | `BANDS.highPusher.volume` |
| High Pusher card spend | $5K/ref/mo | `BANDS.highPusher.spend` |
| High Pusher card sales | $1K/ref/yr | `BANDS.highPusher.cardActivityAnnual` |
| Slider range | 0–500 | slider element |

If we ever get real network data, we should recalibrate the two bands against actual user behavior on Tria. The current values are reasoned estimates, not measured.

---

## Open questions for the team

1. **Slider range** — 500 max is fine for most KOLs, but a top-tier ambassador with a 2,000-person community is at $130K × 4 = $522K/year on the Average band. Visitors can type a number past 500, but the slider stops there. Should we extend?
2. **Band labels** — "Average Pusher / High Pusher" reads aggressive (intentional). If a partner pushes back that "Pusher" sounds too sales-y, alternates are "Engaged / Power" or "Standard / Heavy."
3. **Indirect commission in calc** — we excluded it for simplicity. Should there be a toggle to add it ("with N sub-ambassadors")? Risk: undercuts the simplicity of the single-slider UX.
4. **Tier-jump visualization** — going from Silver 2 (25 refs) to Gold (30 refs) is a significant rate jump. Could we visually mark that on the slider? Currently the tier badge just changes label.
5. **"Average" vs. "Median"** — the headline currently says Average. Median might be more honest if our distributions are skewed. Probably doesn't matter for visitor perception.

---

## Source of truth

- **Tier rates and thresholds:** `/Users/ronaldtatoruiz/tiers.md` (ambassador-side Season 3 spec). Specifically lines 60–66 (tier taxonomy) and lines 47–54 (commission regime rule table).
- **Tria futures taker fee:** 0.05% of volume. Per product team direction; not in the spec doc but agreed in the previous iteration of this work.
- **Active-referral definition:** ≥$1 card spend OR ≥$10K trade volume in rolling 90-day window, counted at depth 1 + depth 2. Defined in the user-side spec at `/Users/ronaldtatoruiz/tiers (1).md`, consumed by the ambassador-side spec.
- **Indirect commission rates:** 10% / 0.05% / 5% locked across all standard-ladder tiers. Spec line 317 / Journey 5.

If any of these change, the spec file (`calculator-spec.md`) lists every formula and constant — finding the impacted cell takes a few seconds.

---

## Why we're not building a chart

ether.fi's calculator has a chart that shows 12-month earnings as three trajectory lines. We considered it. Decision: skip for now. Reasons:

- A chart implies cumulative growth — which suggests new referrals over time, not steady-state rewards on a fixed network. That's a different mental model.
- The three-band cards already communicate floor / median / ceiling at a glance.
- A chart adds visual weight without adding decision-relevant information at this stage.

If we want to add a chart later, it'd be most useful as a "what if your network grows" overlay: slider for monthly new referrals, chart shows cumulative earnings ramp. That's a v2 feature.

---

## End-state

This calculator does one job: convert "how big is my network" into "how much can I earn." It does it with one slider, two projection bands (Average Pusher / High Pusher), and a transparent breakdown of the headline number. Visitors who want product-level detail (Virtual / Signature / Premium pricing, indirect commission mechanics, the higher hierarchy) go to the portal.

If you have feedback on the band assumptions, the slider range, or the copy — flag it. Everything except the tier rates is tunable; the tier rates are locked to the spec.

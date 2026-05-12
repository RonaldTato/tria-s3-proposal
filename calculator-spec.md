# Tria Ambassador Calculator — Implementation Spec

> Drop this entire file into Claude Design (or hand it to an engineer). Every constant, formula, and state transition needed to rebuild the calculator is below. Inspired by ether.fi's affiliate calculator structure, but uses Tria's actual rates and adds a three-band projection.
>
> Live reference: https://ronaldtato.github.io/tria-s3-proposal/calculator.html

---

## 1. Purpose

A public-facing earnings projection tool for ambassadors. The visitor manipulates **one input** (active referrals); the calculator shows **three projection bands** (Conservative / Average / High) for 12-month rewards.

This is deliberately less detailed than an internal commission simulator. The public surface should not require the visitor to know their network's trading volume, card-product mix, or card-spend distribution — all of that is baked into the three bands.

---

## 2. Interaction model

**Inputs:**
- `activeReferrals` — integer ≥ 0. Slider 0–500 with typeable numeric input that can exceed slider max.

**Auto-derived state:**
- `tier` — one of `Bronze 1 / Bronze 2 / Silver 1 / Silver 2 / Gold`, computed from `activeReferrals` using the threshold table below.

**Outputs:**
- Headline annual reward (Average band × 12)
- Three band totals (annual): Conservative / Average / High
- Stream breakdown for the Average band (3 rows: Futures / Card-spend / Card-sale with %)
- Tier badge

**Behavior:**
- Slider and numeric input stay in sync. When the typed value exceeds slider max, the slider clamps to max but the calculation uses the typed value (no upper cap on calculation).
- Tick marks (suggested): `0 / 25 / 50 / 100 / 250 / 500`. Clicking a tick jumps slider to that value.
- All output values recompute on every input change.

---

## 3. Constants

### 3.1 Tier ladder

| Tier | Min refs | Max refs | Card sales % | Card spend % | Futures fee % |
|---|---:|---:|---:|---:|---:|
| Bronze 1 | 0 | 2 | 20.00% | 0.00% | 3.00% |
| Bronze 2 | 3 | 9 | 22.00% | 0.05% | 5.00% |
| Silver 1 | 10 | 19 | 25.00% | 0.10% | 8.00% |
| Silver 2 | 20 | 29 | 27.00% | 0.15% | 11.00% |
| Gold | 30 | ∞ | 30.00% | 0.25% | 15.00% |

### 3.2 Tria platform constants

| Constant | Value | Notes |
|---|---|---|
| Tria futures taker fee | 0.05% (= 0.0005) | Applied to trading volume to get fee Tria collects |

### 3.3 Per-referral assumptions by band

Each band represents a different per-referral activity profile. All amounts are **per referral**.

| Band | Trading volume / month | Card spend / month | Card-sale activity / year |
|---|---:|---:|---:|
| Conservative | $25,000 | $50 | $20 |
| Average | $250,000 | $500 | $70 |
| High | $1,000,000 | $1,500 | $200 |

**Rationale for `cardActivityAnnual`:** This number absorbs the product mix (Virtual $20 / Signature $90 / Premium $225) into one annual gross figure per referral. Conservative ≈ one Virtual card per ref/year + occasional reactivation. Average ≈ one Signature per ref/year + extras. High ≈ one Premium per ref/year + upgrades.

### 3.4 Indirect commission (not in calculator math; FAQ-only)

| Stream | Indirect rate | Applied to |
|---|---|---|
| Card sales | 10% | Card-sale fee Tria collects |
| Card spend | 0.05% | Card spend volume |
| Futures | 5% | Futures fee Tria collects |

**Indirect is ONE level deep — not an infinite downline.** Standard tiers (Bronze 1 → Gold) earn indirect commission only on transactions from the direct customers of ambassadors they sponsored. The chain stops there: a sponsored ambassador's own sponsored ambassadors' customers do **not** pay indirect to the original ambassador. Higher hierarchy operators (Platinum, Diamond) earn deeper revenue via a separate spread-to-max-tier mechanism, but that's outside this calculator's scope.

Indirect commission is explicitly **excluded from the calculator's headline math** to keep the slider focused on direct activity. It's mentioned in an FAQ disclosure as a "compounder" — rough heuristic: 5 sponsored ambassadors mirroring your network ≈ 2× annual run-rate.

---

## 4. Formulas

### 4.1 Tier resolution

```js
const TIERS = [
  { id: 'b1', name: 'Bronze 1', min: 0,  max: 2,        cardSales: 0.20, cardSpend: 0.0000, futures: 0.03 },
  { id: 'b2', name: 'Bronze 2', min: 3,  max: 9,        cardSales: 0.22, cardSpend: 0.0005, futures: 0.05 },
  { id: 's1', name: 'Silver 1', min: 10, max: 19,       cardSales: 0.25, cardSpend: 0.0010, futures: 0.08 },
  { id: 's2', name: 'Silver 2', min: 20, max: 29,       cardSales: 0.27, cardSpend: 0.0015, futures: 0.11 },
  { id: 'gd', name: 'Gold',     min: 30, max: Infinity, cardSales: 0.30, cardSpend: 0.0025, futures: 0.15 },
];

const tierFor = (refs) => TIERS.find(t => refs >= t.min && refs <= t.max);
```

### 4.2 Band data

```js
const TRIA_FUTURES_FEE = 0.0005;  // 0.05% taker fee

const BANDS = {
  conservative: { volume: 25000,   spend: 50,   cardActivityAnnual: 20  },
  average:      { volume: 250000,  spend: 500,  cardActivityAnnual: 70  },
  high:         { volume: 1000000, spend: 1500, cardActivityAnnual: 200 },
};
```

### 4.3 Monthly earnings per band

```js
function monthlyForBand(refs, tier, band) {
  const futures = refs * band.volume               * TRIA_FUTURES_FEE * tier.futures;
  const spend   = refs * band.spend                * tier.cardSpend;
  const sales   = refs * (band.cardActivityAnnual / 12) * tier.cardSales;
  return {
    futures,
    spend,
    sales,
    total: futures + spend + sales,
  };
}
```

### 4.4 Annual run-rate

```js
const annual = monthlyTotal * 12;
```

That's the entire math.

---

## 5. Display structure (top → bottom)

| # | Element | Content |
|---|---|---|
| 1 | Hero | Eyebrow "For Ambassadors" / headline "How much can you earn?" / sub explaining the three-band model |
| 2 | Slider row | `[Big number] active referrals` on left · `[Tier pill]` on right · slider below · tick marks underneath |
| 3 | Headline | Label "Projected 12-month rewards · Average" · large gradient number · sub "USDC / USDT, claimable in-app · paid in stablecoin" |
| 4 | Three bands | Side-by-side cards: Conservative · Average (featured/emphasized) · High · each shows annual total + monthly equivalent |
| 5 | Breakdown | Single panel showing where the Average band comes from: Futures fees / Card spend / Card sales, each with % of total |
| 6 | Higher-hierarchy callout | Single sentence, no specifics: "There is a higher hierarchy above Gold — accessible by accelerated climb or direct invitation. Its terms are not published." |
| 7 | FAQ disclosures | Three collapsible items: "What's Conservative / Average / High?" + "What about sponsoring sub-ambassadors?" + "How are rates applied?" |
| 8 | Footnote | Disclaimer: estimates based on per-referral assumptions, actual results vary |

---

## 6. States to handle

| Input state | Expected output |
|---|---|
| `refs = 0` | Tier = Bronze 1. All band totals = $0. Headline = $0. |
| `refs = 30` | Tier = Gold. Numbers match the defaults shown in the spec. |
| `refs > 500` (typed) | Slider clamps to 500; calculation uses typed value (no cap). |
| `refs ≥ 30` | Tier remains Gold. No projection above Gold rates. |
| Numeric input cleared | Treat as 0 (graceful fallback). |
| Slider dragged | Numeric input updates in sync. |
| Numeric input changed | Slider updates (clamped to max). |
| Tick clicked | Slider + numeric input both jump to that value. |

---

## 7. Sanity-check values

The implementation should produce these exact numbers at the listed inputs:

| Active refs | Tier | Conservative (12mo) | Average (12mo) | High (12mo) |
|---:|---|---:|---:|---:|
| 0 | Bronze 1 | $0 | $0 | $0 |
| 5 | Bronze 2 | $61 | $467 | $1,765 |
| 15 | Silver 1 | $264 | $2,153 | $8,220 |
| 25 | Silver 2 | $570 | $4,823 | $18,525 |
| 30 | Gold | $900 | $7,830 | $30,150 |
| 100 | Gold | $3,000 | $26,100 | $100,500 |
| 250 | Gold | $7,500 | $65,250 | $251,250 |
| 500 | Gold | $15,000 | $130,500 | $502,500 |

If the rebuild produces these numbers ± rounding, the math is correct.

Note the **jump from Silver 2 → Gold at 30 refs**: Conservative goes from ~$570 → $900, Average from ~$4,823 → $7,830, High from ~$18,525 → $30,150. The rate uplift across all three streams at the Gold threshold is significant — design should make the tier badge visibly upgrade when crossing 30.

---

## 8. Tone & content rules (must follow)

- **Do not mention** card product names (Virtual / Signature / Premium) on the public surface. Each band's `cardActivityAnnual` absorbs the product mix.
- **Do not mention** Platinum, Diamond, or any tier above Gold by name. The "higher hierarchy" callout stays vague.
- **Do not publish** any rate above the Gold row.
- **Do not show** indirect commission rates on the calc surface itself; FAQ-only.
- **Do not include** competitor comparisons (no ether.fi, no other affiliate platforms).
- Payout copy: "USDC / USDT, claimable in-app · paid in stablecoin" — no Merkle/atomic/idempotent/proof language.

---

## 9. Recommended copy

### Hero
- Eyebrow: `For Ambassadors`
- H1: `How much can you earn?`
- Sub: `One slider. Three projections built from real Tria rates. Modeled on per-referral activity — three bands let you see the floor, the median, and the ceiling without modeling product-level detail.`

### Headline
- Label: `Projected 12-month rewards · Average`
- Sub: `USDC / USDT, claimable in-app · paid in stablecoin`

### Higher-hierarchy callout
> These are the public ladder rates, capped at Gold. There is a higher hierarchy above it — accessible by accelerated climb or direct invitation. Its terms are not published.

### FAQ disclosures
1. **What's Conservative / Average / High?** — show per-band assumptions table + one paragraph on which network shape maps to which band.
2. **What about sponsoring sub-ambassadors?** — explain indirect commission (10% / 0.05% / 5%), say it compounds on top of the slider math, roughly 2× annual run-rate per 5 subs.
3. **How are rates applied?** — short explainer per stream: card sales % × card price · card spend % × spend volume · futures % × Tria taker fee (0.05% × volume).

### Footnote
> Projections are estimates based on per-referral activity assumptions. Actual earnings vary with your network's behavior, the rate of card activations, and trading conditions. Rates and tier thresholds from the approved Tria Season 3 ambassador-tiers spec. Calculator does not project the higher hierarchy above Gold.

---

## 10. Visual hints (optional — design owns this)

- Dark background (near-black, not pure #000)
- Headline number uses a gradient fill (suggestion: pink → gold) to draw the eye
- Three band cards in a row; emphasize the Average card with a glowing border or soft fill
- Tier pill in muted gold; the slider thumb in pink
- Sliders ≠ scroll-jacked; keep tick marks legible
- Mobile: stack the three bands vertically when viewport < 600px

End of spec.

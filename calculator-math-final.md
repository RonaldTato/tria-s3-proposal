# Tria Ambassador Calculator — Final Math Logic + Rationale

> Single-source-of-truth doc for the team to cross-check. Every constant, every formula, every assumption — and *why* — laid out for review against the Season 3 spec (`/Users/ronaldtatoruiz/tiers.md`) and the commission simulation spreadsheet (`Copy of Commissions Testing.xlsx`).

**Live calculator:** https://ronaldtato.github.io/tria-s3-proposal/landing/#calculator

---

## 1. The model in one paragraph

The calculator takes a **single input** (active referrals) and emits a **single output** (12-month projected earnings) under two band assumptions about how active each referral is on Tria. Per-referral activity drives three commission streams — futures trading fees, card spend, and card-subscription sales — each multiplied by the tier rate that the referral count triggers. The visitor sees the headline number for the **Average Pusher** band; the **Heavy Pusher** band sits alongside as an upper-bound projection.

---

## 2. Constants

### 2.1 Tier ladder (source: spec lines 60–66 of `tiers.md`)

| Tier | Active refs (min) | Active refs (max) | Card sales % | Card spend % | Futures fee % |
|---|---:|---:|---:|---:|---:|
| Bronze 1 | 0 | 2 | 20.00% | 0.00% | 3.00% |
| Bronze 2 | 3 | 9 | 22.00% | 0.05% | 5.00% |
| Silver 1 | 10 | 19 | 25.00% | 0.10% | 8.00% |
| Silver 2 | 20 | 29 | 27.00% | 0.15% | 11.00% |
| Gold | 30 | ∞ | 30.00% | 0.25% | 15.00% |

The calculator caps at Gold. The **higher hierarchy** (Platinum, Diamond) exists in the spec but is intentionally **not projected on the calculator** — those tiers are invite-only, their rates aren't published, and they earn via a different mechanism (spread-to-max-tier rather than locked ladder rates).

### 2.2 Tria platform constants

| Constant | Value | Source |
|---|---|---|
| Tria futures taker fee | **0.05% of trading volume** | Product team direction (not in spec doc) |
| Avg card price (implicit) | **~$100** | Middle ground between Virtual ($20), Signature ($90), Premium ($225); used only as a back-of-envelope for translating `cardActivityAnnual` into a cards-per-month rate, **not in the math itself** |

### 2.3 Active-referral definition (source: spec, consumed from user-side tiers.md)

A referral counts as "active" if, in the rolling 90-day window, they have either:
- ≥ $1 cumulative card spend, **OR**
- ≥ $10K cumulative trading volume

Counted at **depth 1 + depth 2** (ambassador-side counting symmetry). The calculator doesn't compute this — it just takes "active referrals" as the input from the visitor.

### 2.4 Per-referral assumptions by band

Both bands assume the referral is actively engaged on Tria. No "casual user" floor.

| Per referral (monthly) | Average Pusher | Heavy Pusher |
|---|---:|---:|
| Trading volume | $500,000 | $2,000,000 |
| Card spend | $2,000 | $5,000 |
| Card-subscription sales (annual) | $200 | $1,000 |

**Rationale for `cardActivityAnnual`:**
- Represents gross dollar value of cards your network buys per year (new issuance + upgrades + replacements).
- $200/year ÷ $100 avg card price ≈ 2 cards/year per referral.
- Calibrated against ambassador throughput: a 30-ref Gold network at the Average Pusher band → **~5 cards/month total** (30 × 200 ÷ 12 ÷ $100). Calibrated against your direct feedback ("4–5 cards/month average").
- Heavy Pusher: 30 × 1000 ÷ 12 ÷ $100 = ~25 cards/month at 30 refs (within your stated 20–30/month range).

---

## 3. The math

### 3.1 Per-stream monthly commission formula

For each stream, the formula is **identical** in shape:

```
streamMonthly = refs × (per-ref base) × tier.rate
```

Specifically:

```js
const futuresMonthly = refs × band.volume   × TRIA_FUTURES_FEE   × tier.futures;
const spendMonthly   = refs × band.spend                          × tier.cardSpend;
const salesMonthly   = refs × (band.cardActivityAnnual / 12)      × tier.cardSales;
```

Where:
- `refs` = active referrals (user input)
- `band.volume`, `band.spend`, `band.cardActivityAnnual` = the per-referral assumptions from §2.4
- `TRIA_FUTURES_FEE` = 0.0005 (Tria's 0.05% taker fee on volume)
- `tier.cardSales`, `tier.cardSpend`, `tier.futures` = the tier rates from §2.1

### 3.2 Total and annualization

```js
const monthlyTotal = futuresMonthly + spendMonthly + salesMonthly;
const annualTotal  = monthlyTotal × 12;
```

That's the entire math. No multiplier, no fudge factor, no hidden adjustment.

### 3.3 What gets displayed

| Display element | Source |
|---|---|
| **Headline** "$X,XXX" | `annualTotal` for the Average Pusher band |
| **Average Pusher card** (featured) | `annualTotal` and `monthlyTotal` for Average Pusher band |
| **Heavy Pusher card** | `annualTotal` and `monthlyTotal` for Heavy Pusher band |
| **Stream breakdown** | The three `streamMonthly` values for the **Average Pusher band only**, with each stream's % share of the monthly total |
| **Tier pill** | `tierFor(refs)` — looks up which tier the referral count triggers |

---

## 4. Worked example — line by line at default load (30 refs, Gold tier)

### Average Pusher band

```
Stream         Formula                                       = Monthly
─────────────────────────────────────────────────────────────────────
Futures        30 × $500,000 × 0.0005 × 0.15                 = $1,125.00
Card spend     30 × $2,000             × 0.0025              =    $150.00
Card sales     30 × ($200 ÷ 12)        × 0.30                =    $150.00
                                                              ─────────
Monthly total                                                  $1,425.00
Annual total                                                  $17,100.00
```

### Heavy Pusher band

```
Stream         Formula                                       = Monthly
─────────────────────────────────────────────────────────────────────
Futures        30 × $2,000,000 × 0.0005 × 0.15               = $4,500.00
Card spend     30 × $5,000               × 0.0025            =   $375.00
Card sales     30 × ($1,000 ÷ 12)        × 0.30              =   $750.00
                                                              ─────────
Monthly total                                                  $5,625.00
Annual total                                                  $67,500.00
```

### Composition of the Average Pusher headline (used for the breakdown chart)

| Stream | Monthly | % of total |
|---|---:|---:|
| Futures fees | $1,125 | 79% |
| Card spend | $150 | 11% |
| Card sales | $150 | 11% |
| **Total** | **$1,425** | **100%** |

---

## 5. Scaling sanity table (Gold tier, fixed band assumptions)

| Active refs | Tier | Average Pusher (annual) | Heavy Pusher (annual) |
|---:|---|---:|---:|
| 0 | Bronze 1 | $0 | $0 |
| 5 | Bronze 2 | $1,030 | $4,250 |
| 15 | Silver 1 | $4,710 | $19,050 |
| 25 | Silver 2 | $10,500 | $42,000 |
| 30 | Gold | $17,100 | $67,500 |
| 100 | Gold | $57,000 | $225,000 |
| 250 | Gold | $142,500 | $562,500 |
| 500 | Gold | $285,000 | **$1,125,000** |

The implementation should produce these numbers ±$1 of rounding at every row.

---

## 6. What's intentionally NOT in the math

These are flagged in the FAQ disclosures below the calculator but excluded from the slider math:

### 6.1 Indirect commission (sponsoring sub-ambassadors)

The locked indirect rates per the spec (§4.5 of the consolidated commission rule):
- **10%** of card-sale fees
- **0.05%** of card spend
- **5%** of futures fees

These rates pay you on transactions from your **sponsored ambassador's direct customers** — one level deep, **not infinite downline**. The chain stops there for standard tiers.

**Why excluded from the headline slider:**
- Keeps the calculator focused on what *you* can build directly.
- A visitor moving the slider should see the impact of their own network, not their sponsored ambassadors' networks.
- Indirect is communicated in two places:
  1. Section 4 of the landing (the "Indirect, locked rate" block with the bar-chart compound visualization)
  2. The "What about sponsoring sub-ambassadors?" FAQ disclosure inside the calc card

### 6.2 Compounding math (for the team's cross-check)

If the team wants to *include* sub-ambassadors in a future calc, here's the math.

Given:
- `you.directMonthly` = your direct run-rate at your current tier
- Each sponsored ambassador mirrors your network shape (Average Pusher, same refs, same tier)

Each sub adds the following indirect commission to your run-rate per month:

```
indirectPerSub = refs × volume   × TRIA_FUTURES_FEE   × INDIRECT.futures
               + refs × spend                          × INDIRECT.cardSpend
               + refs × (cardActivityAnnual ÷ 12)      × INDIRECT.cardSales

where INDIRECT = { cardSales: 0.10, cardSpend: 0.0005, futures: 0.05 }
```

Worked at 30 refs, Average Pusher, Gold tier:

```
Indirect futures:     30 × $500,000   × 0.0005 × 0.05  = $375 / mo
Indirect spend:       30 × $2,000              × 0.0005 = $30  / mo
Indirect card sales:  30 × ($200 ÷ 12)         × 0.10   = $50  / mo
                                                          ─────
Total per sub                                            $455 / mo
```

**Lift ratio = $455 / $1,425 = 31.9%** → each sub adds ~32% to your run-rate (assuming mirror network).

Why ~32%? It's the weighted average of the indirect/direct ratios across streams at Gold:

| Stream | Indirect/Direct | Weight (Avg Pusher comp) | Contribution |
|---|---:|---:|---:|
| Card sales | 10% ÷ 30% = 0.333 | 11% | 0.037 |
| Card spend | 0.05% ÷ 0.25% = 0.20 | 11% | 0.022 |
| Futures | 5% ÷ 15% = 0.333 | 79% | 0.263 |
| **Total** | | | **0.322** |

So:

| Sponsored subs | Run-rate multiplier |
|---:|---|
| 0 | 1.00× |
| 1 | 1.32× |
| 3 | 1.96× |
| 5 | 2.60× |
| 10 | 4.20× |

**Note on lower tiers:** because indirect rates are *locked* (10%/0.05%/5% regardless of your tier), but direct rates climb with tier, the lift ratio is actually **higher at lower tiers**. A Silver-1 ambassador (25% direct on card sales) sponsoring a Silver-1 sub gets a card-sales lift ratio of 10/25 = 40%, not 33%. Sponsoring is most economically valuable for ambassadors who are still climbing.

### 6.3 Higher hierarchy (Platinum / Diamond)

The calculator caps at Gold. Above Gold:
- Platinum and Diamond exist as exclusive tiers, granted only by invitation.
- They earn through a **spread-to-max-tier** mechanism (Platinum fills the spread up to 33%, Diamond up to 50%) — explained in the spec, verified against the Excel test case `Default + User Referral 3010202` rows 4–10.
- The calculator does **not** project this. The "Higher hierarchy above it — accessible by accelerated climb or direct invitation. Its terms are not published." callout below the calc references this without exposing rates.

---

## 7. Cross-checks against the spreadsheet

The Excel file `Copy of Commissions Testing.xlsx`, sheet `Default + User Referral 3010202`, rows 4–10, verifies the commission DISTRIBUTION logic the calculator's per-ref / per-stream model is built on:

| Chain position | Earns on Bronze transactor's $100 spend | Math |
|---|---:|---|
| Bronze (self) | $20 | 20% direct |
| Silver (1-up) | $10 | 10% indirect (locked) |
| Gold (2-up) | **$0** | Standard tiers cap at 1 level of indirect |
| Platinum | $3 | 33% − 30% spread |
| Diamond | $17 | 50% − 33% spread |
| Referrer (top) | $2 | 2% top-rate |

The calculator's `streamMonthly` formula in §3.1 is a clean expression of the **transactor + direct-upline** portion of this rule. It deliberately does not model the spread mechanics for higher hierarchy or the locked-indirect for sponsored ambassadors — both are excluded by design (see §6).

---

## 8. Edge cases and behavior

| Input state | Expected behavior |
|---|---|
| `refs = 0` | All streams = $0. Headline = $0. Tier badge = Bronze 1. |
| `refs = 1` | Bronze 1 tier. Annual ≈ $69 (Average Pusher) / $290 (Heavy Pusher). |
| `refs = 30` | Tier jumps to Gold. The "tier badge upgrade" should be visually distinct. |
| `refs > 500` (slider max) | Slider caps at 500, but **typed input can exceed** — the math will compute correctly for any positive integer. |
| Numeric input cleared | Treated as 0. Graceful fallback. |
| Tick clicked | Jumps slider + input to that value, recomputes. |

---

## 9. What changes if a constant changes

If the team adjusts a constant, here's the blast radius:

| Constant | Effect on calculator | Effect on landing copy |
|---|---|---|
| Tier rate change (e.g., Gold futures 15% → 18%) | Linear scale on that stream | "Up to 15%" copy in the streams-chart needs an update too |
| Tria futures fee change (0.05% → 0.07%) | Futures stream scales by 1.4× | Footer text + FAQ "How are rates applied" mention 0.05% — update both |
| Active-referral threshold change (e.g., 30 → 25 for Gold) | Tier pill triggers earlier | Public ladder table on the landing — Gold row's "Active referrals" cell |
| Band assumption change (e.g., Avg Pusher vol $500K → $750K) | All Average Pusher totals scale linearly | FAQ "What's Average Pusher vs Heavy Pusher?" table |
| Indirect rate change (10/0.05/5%) | Affects §6.2 lift math only — not the slider math | Section 4 of the landing (the three big rate tiles) + the FAQ disclosure |
| Card-product price change (Virtual $20 → $25, etc.) | None on calc (prices are baked into `cardActivityAnnual`) | "Card-subscription sales" FAQ paragraph mentions $20/$90/$225 — update there |

---

## 10. Recommended team review questions

For the team to validate before sign-off:

1. **Tier rates (§2.1)** — do these match the rates engineering will actually ship in Season 3?
2. **0.05% Tria taker fee (§2.2)** — is this the locked production rate, or could it change before launch?
3. **Per-referral assumptions (§2.4)** — calibrated against your "4–5 cards/month" feedback at 30 refs Gold. Does the team agree these are realistic vs. aggressive vs. conservative?
4. **Sanity table (§5)** — at 500 refs Gold, Heavy Pusher reads $1.125M/yr. Is the team comfortable publishing this projection?
5. **Indirect framing (§6.2)** — currently the landing's "How it adds up" block shows 1.2× / 1.6× / ~2.0× for 1/3/5 subs. The correct mirror-network math is 1.3× / ~2.0× / ~2.6× (+32% per sub). Bump to actual numbers, or keep conservative messaging?
6. **Higher hierarchy (§6.3)** — confirm the calculator should NOT project Platinum/Diamond rates.

End of doc.

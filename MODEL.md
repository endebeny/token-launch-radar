# Token Launch Radar — the radar-2 model

The board this repository builds is scored by **radar-2**, a model written for
this entry. It reads one sealed snapshot — every protocol DefiLlama tracks,
fetched 2026-11-14 — and ranks the protocols that have no token by the implied
value of the token they may one day issue. This file is the model's full
specification: the formula, the constants, the calibration, a worked example,
and the limits. Nothing here is a price, and nothing here is a promise that a
token is coming.

## The snapshot

- Source: DefiLlama `https://api.llama.fi/protocols`, fetched
  2026-11-14T01:18:33.201Z. Snapshot date: 2026-11-14.
- 758 protocols in the snapshot. 40 of them are on the watchlist: every
  protocol with at least $50,000,000 of TVL and no token on record. The
  watchlist is the floor, never a ceiling.
- 342 protocols in the snapshot record no token at all. 186 of them are
  rankable pre-token protocols below the $50M floor. The board covers all
  186, so the board is 226 rows: 40 watchlist + 186 early.
- 24 protocols with a live token are set aside. They are shown with the
  market's own values, never mixed into the ranked board.
- A protocol counts as pre-token when its index entry records no market cap,
  no CoinGecko id, and no symbol. No protocol name is ever special-cased:
  the same function scores every row.

## The formula

For a pre-token protocol, radar-2 computes:

```
base         = tvl_usd x 0.25
growth       = clamp(change_7d, -60, 120)          (per cent)
growthMult   = 1 + (growth / 100) x 0.75
maturity     = 0.55 + 0.45 x sqrt(min(age_days, 1095) / 1095)
chainsMult   = 1 + 0.03 x min(chains - 1, 5)
implied      = max(1, round(base x growthMult x maturity x chainsMult))
```

`age_days` is the days since `listed_at`. When `listed_at` is unknown, the
model uses 365 days. `chains` is the number of chains the protocol runs on.

## Why each piece

- **Base multiple 0.25.** TVL is the size of the economy a token would
  price. A token's fully diluted value has historically landed near a
  fraction of the value its protocol secures, so the model takes a quarter
  of TVL as the starting point and lets the multipliers move it. The
  quarter is a deliberately conservative anchor: it keeps the board from
  promising a token worth more than the economy behind it.
- **Growth weight 0.75.** Seven-day TVL growth is the momentum a launch
  would ride, but a single week is noisy. The growth is clamped to
  -60..120 per cent so one freak week cannot dominate, and it is weighted
  at 0.75 so momentum matters without swamping size and age. A protocol
  growing 20% in a week scores 1.15 on this multiplier; one shrinking 20%
  scores 0.85.
- **Maturity, square root.** Age with no token is the pent-up value of a
  launch long deferred. The square root gives diminishing returns: the
  fifth year without a token adds less than the first. The floor is 0.55,
  so even a brand-new protocol keeps more than half its base value, and
  the cap is 1.00 at three years (1095 days), so age alone can never more
  than double the base. Unknown age counts as one year (365 days).
- **Chain weight 0.03.** Each extra chain is another market a token would
  launch into, worth 3% each, capped at five extra chains (1.15 at most).
  The cap stops a protocol listed on twenty chains from outrunning its
  own economy on breadth alone.

## Worked example: Steakhouse Financial, rank 1

Steakhouse Financial is the board's top row, with an implied value of
$675,083,656. Its snapshot figures:

| figure | value |
|---|---|
| TVL | $2,750,000,000 |
| seven-day growth | -2.19% |
| age with no token | 547 days (1.5y) |
| chains | 11 |

Step by step:

- base = 2,750,000,000 x 0.25 = 687,500,000
- growth = -2.19 (inside the -60..120 clamp), so
  growthMult = 1 + (-2.19 / 100) x 0.75 = 0.9835
- maturity = 0.55 + 0.45 x sqrt(547 / 1095) = 0.55 + 0.45 x 0.7068 = 0.8669
- chainsMult = 1 + 0.03 x min(11 - 1, 5) = 1 + 0.03 x 5 = 1.15
- implied = round(687,500,000 x 0.9835 x 0.8669 x 1.15) = 675,083,656

The dashboard's detail view recomputes this row from the snapshot's own
figures and shows the same number the board carries, so any reader can
check the arithmetic.

## The farm plans

Every board row carries a farm plan of six steps. The steps are derived
from the row itself — its category, its chains, its TVL, and its age — not
written by hand per protocol:

1. A fresh wallet that holds only the funds to be farmed, seed phrase
   offline. Cost: under $3 of gas for the first transaction. Time:
   5 minutes.
2. A deposit sized to the protocol (around $150 for the larger ones),
   on its largest chain, held in the position. Cost: the deposit plus
   about $8 of gas. Time: 15 minutes to set up.
3. One qualifying action each week — a deposit, a borrow, a swap, or a
   vote — matched to the protocol's category. Cost: about $2 of gas a
   week. Time: 10 minutes a week.
4. Hold the position for at least 30 days, so the activity is counted
   in more than one snapshot. Cost: the deposit stays in the position.
   Time: 30 days.
5. Claim points or rewards only from the protocol's own page, before
   withdrawing. Cost: a claim transaction's gas. Time: 10 minutes.
6. A closing step specific to the category (for lending: repay and
   withdraw; for a DEX: remove liquidity; and so on). Cost and time are
   stated on the step.

## The dashboard

`dashboard/` renders the board with no network. It offers a search box,
category and chain filters, a minimum-TVL filter, a watchlist-only
toggle, pre-token and live-token tabs, sortable columns (rank, name, TVL,
seven-day growth, age, implied value), a detail view per protocol with
the full model breakdown, and a model sandbox: sliders for the growth
weight (0–1.5, default 0.75) and the chain weight (0–0.06, default 0.03)
that recompute implied values and re-rank the page live. The sandbox
never changes `board.json`; the default view equals the submitted board.

## What the model does not do

- It does not predict a price. Implied value says what a token would be
  worth if the protocol kept running as it is, never that one is coming.
- It does not use anything outside the snapshot: no live prices, no
  social signals, no team or investor information.
- It does not rank live-token protocols against pre-token ones. The
  live-token tab is a separate view with market values.
- It is deterministic: `node main.ts` run twice on the same snapshot
  writes byte-identical `board.json`, `data.js`, and `dossier.json`.

## Reproduce it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

The run writes `out/board.json` (226 rows), `out/data.js`, the dashboard,
the screenshots, and `out/dossier.json`. The board's top ten rows carry a
combined implied value of $2.17B; the median pre-token protocol is 1.3
years old with no token.

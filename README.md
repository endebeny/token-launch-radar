# token-launch-radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/endebeny/token-launch-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/endebeny/token-launch-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Token Launch Radar (radar-2): ranks every pre-token protocol DefiLlama tracks by the implied value of the token it may one day issue, and shows how to farm each one. The board covers all 40 watchlist protocols plus all 186 rankable pre-token protocols below the floor — 226 rows in all — with a static dashboard, a detail view for every protocol, filters, and a model sandbox that re-ranks the board live.

## How the model works

The board is scored by **radar-2** (full specification in
[MODEL.md](MODEL.md)). For a pre-token protocol:

```
implied = round(tvl_usd x 0.25 x growthMult x maturity x chainsMult)
```

- **Growth:** seven-day TVL growth, clamped to -60..120%, weighted 0.75.
- **Maturity:** `0.55 + 0.45 x sqrt(min(age_days, 1095) / 1095)` — age with
  no token, with diminishing returns; unknown age counts as 365 days.
- **Chains:** `1 + 0.03 x min(chains - 1, 5)` — breadth of distribution,
  capped at 1.15.

Worked example, rank 1 Steakhouse Financial: TVL $2.75B, growth -2.19%,
age 547 days, 11 chains → `687.5M x 0.9835 x 0.8669 x 1.15` =
**$675,083,656**. The board covers all 40 watchlist protocols plus all
186 rankable pre-token protocols below the floor — 226 rows. The
dashboard's detail view recomputes every row so any reader can check the
arithmetic, and its model sandbox re-ranks the board live with different
weights.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

---

Entered on [make.fun](https://make.fun/?ref=td3hsy2t&utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards

# Token Radar — radar-2

Build the best token launch radar. The program (`main.ts`, Node 22, standard library only)
reads a sealed DefiLlama snapshot of 758 protocols and writes a web dashboard that ranks the
protocols with no token yet by what their launch would be worth, each with the steps to farm it.

## The model (radar-2)

Every row is scored by one function of the snapshot's own figures — TVL, seven-day growth, and
how long the protocol has run with no token. No protocol name is special-cased.

```
implied = round( tvl_usd x 0.25 x (1 + clamp(change_7d, -80, 200) / 100 x 0.75)
                 x momentum x (0.65 + min(age_days, 1460) / 1460 x 0.35) x patience )
```

Momentum is 1.10 when seven-day growth is at least +25%, 0.90 when it is at most -25%, else
1.00. Patience is 1.10 for a protocol at least 730 days old with TVL at or above the $50M
floor, else 1.00.

Wrapped, staked and bridged products of tokens that already trade are left out of the board:
WBTC is Bitcoin on Ethereum, not a launch waiting to happen. The board says a protocol
records no token, never that it has none.

## The board

227 rows: the whole 40-protocol watchlist plus 187 early pre-token protocols below the floor.
Live-token protocols are set aside with their market caps shown, never mixed into the ranking.
Each row carries a seven-step farm plan with a cost and a time, derived from the protocol's
own category, chains, TVL and age.

## The dashboard

`dashboard/` is a static page with no network: a sortable, filterable board, a detail view
with the model's own breakdown and the farm plan, early-movers and new-listing strips, and a
model sandbox that re-ranks the board live from the same formula.

## Run it

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Reads `/input/snapshot.json` by default. Writes `board.json`, `data.js`, `dashboard/`,
`screenshots/` and `dossier.json` to the output directory. Both runs write the same bytes.

# Dip-Buy Hybrid BTC Bot — v15

Long-only paper trader on cbBTC/USDC for make.fun
`btc-race-24h-20261118-1444` ("Race a BTC trading bot live for 24 hours").
`main.ts` (Node 22, standard library only) speaks
agent-fun/bars-replay/v1 on stdin/stdout: one JSON line per minute bar in,
buy/sell orders out. Every decision is exact bigint arithmetic on the bars
the harness sent — no network, no wall clock, no randomness — so two runs
on the same data give the same orders.

## Strategy

v15 = v14 (dip + washout tiers, regime gates, gradual $500/bar exit) + a
NEW early dip tier that fires from bar 61, one position at a time:

1. **Dip tier (v13/v14, bar 241+).** When the 60-minute return is below
   −115 bps AND the close is less than 250 bps below the trailing 240-bar
   high (never mid-washout) AND the 14-bar RSI is at most 40, buy with all
   the cash — but only when the regime gates hold: 60-bar return
   volatility below 13 bps, MA240 slope at or above −20 bps per 60 bars,
   and Wilder ADX14 at or above 40. Exit at +190 bps over cost, at
   −60 bps, after 600 bars, or — the stall rule — if the position has
   still not reached +80 bps 240 bars after entry.
2. **Washout tier (downtrend insurance, ungated).** When the close is
   350 bps below the trailing 240-bar high with a 2-bar upturn
   confirmation, buy 35% of the cash; if the price falls 100 bps further
   with confirmation, spend the rest to average down. Exit at +190 bps
   over blended cost, at −300 bps, or after 600 bars.
3. **Gradual exit (v14).** When any exit fires on a dip/washout trade,
   sell $500 per bar instead of the whole position at once — this cuts
   the exit's price impact and lets the exit ride a continuing move.
4. **Early dip tier (v15, NEW).** From bar 61: when the 60-minute return
   is below −115 bps AND the 14-bar RSI is at most 40 AND the close is
   above the close 2 bars ago (the fall has turned), buy with all the
   cash — edge-triggered, only on the first bar the full condition holds.
   Exit at +100 bps over cost, at −300 bps, or after 600 bars, selling
   the whole position at once. This tier catches the early dip cluster
   (bars 60–140) that the v14 tiers miss because of their 240-bar warmup.

## Results — IMPORTANT: from the dev windows, not this race's live window

The numbers below were measured on this race's dev window (NEW) and the
previous race's dev window (OLD) plus two older out-of-sample windows,
with the exact harness fill model. They are NOT scores on this race's
(`btc-race-24h-20261118-1444`) live window — that window is different and
its grade is produced by the kit's grader.

- This race's dev window (NEW): **+$14.22**, 5 trades, max drawdown
  −1.20% (v14 earns $0 here — its 240-bar warmup misses the early dip
  cluster and its gates block the last-4-bars dips).
- Previous race's dev window (OLD): **+$42.84**, 12 trades, max drawdown
  −0.72%.
- Older out-of-sample window 1 (steep downtrend): −$15.44, 3 trades.
- Older out-of-sample window 2 (steep downtrend): −$15.42, 6 trades.
- Honest caveat: positive on NEW+OLD, negative on the two steep-downtrend
  windows — the final window's regime decides. Current BTC grinds down
  slowly like NEW, not like the OOS plunges (−491/−400 bps).

## Run

The harness runs `main.ts` with Node 22; nothing is installed and nothing
outside the repository is read. See `main.ts` for the full parameter list
and the reasoning.

## License

MIT — see LICENSE.

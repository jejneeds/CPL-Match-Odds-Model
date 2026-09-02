# CPL-Match-Odds-Model
Ball-by-ball Monte Carlo simulation for CPL match odds
# CPL Match Odds Model

A ball-by-ball Monte Carlo simulation that prices Caribbean Premier League
match odds, built from scratch in Python.

The model simulates a T20 match one delivery at a time, ten thousand times,
and counts how often each side wins. That proportion is the price.

## Result

Backtested on **111 CPL matches (2023-2026)** that the model had never seen.

| Metric | Model | Coin flip |
|---|---|---|
| Brier score | **0.2325** | 0.250 |

Player ratings were rebuilt once per season using only cricket played before
that season began, so the model never predicts a match it learned from.

## How it works

**1. Baselines.** Just under a million balls from seven T20 franchise
competitions (IPL, BBL, CPL, T20 Blast, SA20, ILT20, MLC), filtered to 2022
onward and weighted by recency with a two-year half-life. For each phase —
powerplay, middle overs, death — the pool gives an average runs-per-ball,
wicket rate, and full outcome distribution.

**2. Player factors.** Every batter and bowler is measured against that
baseline per phase. A runs factor of 1.10 means 10% above average; a wicket
factor of 0.80 means 20% less likely to be dismissed.

**3. Shrinkage.** Raw factors from small samples are unreliable, so each is
pulled toward the league average in proportion to how little data supports
it. The shrinkage constant was chosen by out-of-sample testing rather than
by eye — see findings below.

**4. Venue factors.** Grounds are measured relative to *their own
competition*, not the pool, to avoid confusing "this is a low-scoring
ground" with "this is a low-scoring league".

**5. Simulation.** Each ball's outcome distribution is scaled so expected
runs match the combined batter x bowler x venue expectation, then drawn at
random. Innings are simulated ball by ball with strike rotation, wickets and
a chase target.

## Findings

**Shrinkage is not optional.** For death-overs batting, using raw factors
scored *worse* (0.0329) than ignoring player data entirely (0.0309).
Unshrunk small-sample factors are actively misleading.

**Optimal shrinkage varies enormously by what you measure.** Batting runs
settle at K=75-100 balls; bowling wicket rates in the powerplay need
K=4,000 — more than almost any bowler has. Runs are learnable, wickets
barely are.

**Venue wicket factors carry no signal at all.** Extending the search to
"ignore the venue entirely" produced the lowest error in all three phases.
Wickets are too rare for ground effects to rise above sampling noise. That
feature was removed rather than kept.

**A visual estimate of K was three times too high.** Eyeballing the spread
of factors by sample size suggested K≈300; out-of-sample testing gave 100.

## Known limitations

**Under-dispersed innings totals.** Validated against 213 real CPL innings:
the model's mean runs (169.9 vs 166.4) and mean wickets (6.56 vs 6.6) are
close, but it produces 200+ innings at 7.4% against a real 11.7%. Cause is
ball-by-ball independence — real innings cluster into tears and collapses
that independent draws average away. Consequence: extreme outcomes are
underpriced and favourites overpriced in mismatches.

**Suspected chase bias, deliberately unfixed.** Across the backtest the
model expected teams batting first to win 49.3% of the time; they won 41.4%.
That gap could be a real chase advantage the model cannot see, or chance
(z≈1.7 across 111 matches). Rather than fit a correction to the same data
that suggested it, the model is pricing the remaining 2026 season unchanged
so the question can be tested forward.

**No captaincy or intent modelling.** Bowlers rotate in a fixed order, and
batters cannot shift gear in a chase.

**Pool coverage gaps.** Players whose careers are mainly in the PSL, the
Hundred or the BPL are treated as league-average.

## Live prediction log

`cpl_predictions.csv` records timestamped model prices for the remaining
2026 CPL fixtures, alongside a human judgment price recorded before the
model was run, the Betfair price at prediction, the closing price, and the
result. Prices are always logged before the market is consulted.

## Data

Ball-by-ball data from [Cricsheet](https://cricsheet.org).

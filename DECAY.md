# How fast a desk goes stale, measured

The desk claims freshness is a real good. That claim is checkable, so it was checked, on this
product's own builds, before anyone was asked to care. Three goods age on three clocks.

The LIST, which pairs are accepted, ages with the test window: a few sessions against a
2,500-session window barely move the statistics. The STANDINGS, where each spread sits
against its bands, age with daily prices. The CROSSINGS, fresh moves beyond two sigma, are
what a stale desk misses outright.

## Standings

Pooled across 12 month-end reference dates over the trailing year, 3,000 accepted pairs,
about 3,200 observations per lag cell, on the 26 August 2026 build:

| desk age | two-sigma standings still beyond | median z drift |
|---|---|---|
| 1 session | 88.1% | 0.08 sigma |
| 1 week | 76.2% | 0.18 sigma |
| 2 weeks | 68.6% | 0.25 sigma |
| 1 month | 51.9% | 0.39 sigma |
| 2 months | 35.2% | 0.55 sigma |
| 3 months | 28.1% | 0.62 sigma |

A fresh two-sigma standing has even odds of still standing about a month later. The curve is
not a single exponential: marginal standings near 2.0 die in days, deep ones persist, so a
fitted single-rate half-life (38 sessions) overstates the tail and the direct 50 percent
crossing, about 22 to 24 sessions, is the honest summary.

The decay of a standing largely IS mean reversion doing its job. A standing that faded mostly
faded because the spread converged, so the honest reading is opportunity expiry, not data
error.

## List

Measured by re-running the full acceptance test against earlier window endpoints at the build
end:

| desk age | share of today's list already accepted then |
|---|---|
| 1 week | 95.2% |
| 1 month | 86.6% |
| 2 months | 79.5% |
| 3 months | 72.7% |

Corroborated in production: consecutive weekly builds overlap 96.7 percent, turnover about
3.3 percent per build, measured on the real register.

A second, cheaper list measurement was run to afford the 12-anchor pooling: a fixed-lag ADF
with a static hedge. It is disclosed here and NOT adopted, because the simplified test only
reproduces 73.8 percent of the register's acceptance at one session of lag. That gap is
method mismatch at the intercept, the production screen selects its lag by information
criterion, and the same mismatch flattens the slope. Its numbers feed nothing published.

## Crossings

0.047 fresh two-sigma crossings per pair per 5 sessions over the trailing year. A reader one
month behind hears about roughly 0.2 crossings per pair late, by which time about a third
have already converged.

## What this buys the reader of the free sample

The sample in this repository ages on exactly these curves, and the table above says what its
age costs at any date you open it: the list stays mostly true for months, the standings do
not. Averaged over a delivery cycle, a desk refreshed weekly keeps about 85 percent of its
standings readings current, bi-weekly about 76, monthly about 69. Those numbers are read off
the curve, and the measurement re-runs with the builds.

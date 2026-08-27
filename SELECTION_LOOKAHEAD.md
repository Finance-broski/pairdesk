# The screen's forward information, measured without look-ahead

Written 2026-08-25. Everything below is reproducible from `prior.py` against the vintage
`us_2026-08-21_lb2500_n1845_3f61ab`.

This started as a test of one senior-desk criticism, that testing 1.7 million combinations without
an economic prior manufactures noise. It ended somewhere else. The prior turns out to matter less
than the way acceptance was labelled, and the labelling accounts for the entire measured edge.

---

## 1. The question the earlier work never asked

Every out-of-sample figure in the product traces back to one line in `enrich.py`:

    mask = df.coint_95 & df.legs_are_i1
    ev = validate_frame(lp, df, mask)

`coint_95` is decided by `score_pairs` on the whole 2,500-session window. `validate_pair` then holds
back the last 250 of those same sessions and calls them out of sample. They are not. Every pair in
the accepted group was chosen knowing what happened in the year it is then judged on.

`portfolio.select` takes the same frame, so the walk-forward book figures inherit it too. The
"walk-forward" in that name refers to how the spread is estimated, which was fixed by
`lookahead_test.py`, and not to how the pairs were picked, which was never tested.

The distinction is between COMPUTATIONAL look-ahead, fitting a hedge ratio on the whole window and
using it from day one, and SELECTION look-ahead, choosing which pairs to look at using data from the
period you then evaluate. The first was found and fixed. The second was not.

---

## 2. Three nulls, in increasing severity

The old headline was that roughly 55 per cent of acceptances are consistent with chance, computed
against a nominal 5 per cent rate. The strict rule requires both orientations to clear, so its true
size is not 5 per cent and the figure was never anchored to a measurement. All three nulls below
measure it directly, on the same names, through the same code.

**Rotation.** Shift one leg out of alignment. False by construction, and each leg keeps its own
drift and volatility.

| class | observed | null | lift |
|---|---|---|---|
| same industry | 5.57% | 2.00% | 2.79 |
| same sector | 3.71% | 0.86% | 4.33 |
| unrelated | 4.00% | 1.36% | 2.95 |

This null is too easy. Rotation destroys the comovement along with the cointegration, so it asks
whether two unrelated series cointegrate. The real failure mode is two names that plainly move
together and whose spread nonetheless wanders.

**Market factor held fixed.** Keep the SPY component of both legs, resample each leg's own
idiosyncratic returns independently. Correlated by construction, not cointegrated by construction.

| class | observed | null | lift | real corr | synthetic corr |
|---|---|---|---|---|---|
| same industry | 5.77% | 2.51% | 2.30 | 0.434 | 0.219 |
| same sector | 3.50% | 1.92% | 1.83 | 0.284 | 0.228 |
| unrelated | 2.58% | 1.50% | 1.72 | 0.259 | 0.220 |

Still not tight enough. Same-industry names share an industry factor as well as a market factor, so
a pair correlated at 0.43 became a synthetic correlated at 0.22 and the null is understated.

**Joint bootstrap.** Resample the two legs' daily returns as PAIRS, keeping each day's two moves
together and drawing days in a new order. Correlation is preserved to three decimals. Reordering
days destroys every long-run tie. n = 4,000 per class, two seeds.

| block | same industry | same sector | unrelated |
|---|---|---|---|
| 1 day | 2.46, 2.30 | 1.59, 1.15 | 1.44, 1.62 |
| 5 days | 1.93, 1.63 | 1.27, 1.23 | 1.21, 1.23 |
| 10 days | 1.25, 1.29 | 1.02, 1.06 | 0.93, 1.06 |

Block length is a judgement call and it matters. A one-day block destroys volatility clustering,
which real data has and which distorts ADF size. A ten-day block preserves clustering but also
preserves part of the very reversion being tested, so it overstates the null. The truth is inside
the range, and one conclusion survives all of it: **same-industry pairs accept above every version
of the null, and same-sector and unrelated pairs accept inside it from a five-day block upward.**

---

## 3. The control that appeared to rescue the screen

Trade accepted pairs and rejected pairs through the same rule, over the same held-back year, hedge
ratio and standardisation frozen at formation, ten basis points a leg.

| arm | n | mean | positive | correlation |
|---|---|---|---|---|
| accepted | 3,000 | +0.1030 | 53.4% | 0.365 |
| rejected | 3,000 | +0.0004 | 31.2% | 0.306 |
| rejected, correlation matched | 3,000 | −0.0099 | 30.9% | 0.373 |

A gap of eleven log points, not explained by correlation. Repeated era by era it was positive in six
years of six, mean +0.055, and the accepted arm alone was positive in six of six.

That is the number the product would have shipped on.

---

## 4. The clean test, and the inversion

The arms above were labelled from the stored vintage. Recompute the label inside each era instead:
end the formation window before the year being traded, test both orientations on that window alone,
take the finite-sample critical value for that window's own length, and only then trade forward. A
pair may be accepted in one year and rejected in the next, which is the point. 6,000 pairs, six eras.

| era | accept rate | accepted | rejected, matched | gap |
|---|---|---|---|---|
| year 1 back | 4.00% | −0.0010 | −0.0026 | +0.0016 |
| year 2 back | 4.05% | −0.0227 | +0.0041 | −0.0268 |
| year 3 back | 4.43% | −0.0094 | +0.0036 | −0.0130 |
| year 4 back | 4.47% | +0.0190 | +0.0466 | −0.0276 |
| year 5 back | 3.95% | −0.0023 | +0.0325 | −0.0348 |
| year 6 back | 4.28% | −0.0488 | −0.0228 | −0.0259 |

**Mean gap −0.0211, positive in one year of six.** The accepted arm loses money in five of six
years. A t across these six eras computes to −3.92 but should not be quoted as one: adjacent
formation windows share most of a decade of data and most of their names, so the six gaps are not
independent observations. The per-era signs are the honest strength claim; the direction is not in
doubt, the precision of any cross-era t is. Correlation matching is held constant between this and the contaminated run, so the only
thing that changed is where the label came from.

---

## 5. Locating the leak exactly

Take only the pairs accepted IN ERA, on pre-split data alone, and split them by whether the stored
vintage also accepted them. Both groups passed the identical test on identical evidence. The only
difference between them is information from the future.

| group | mean | positive |
|---|---|---|
| the vintage also accepted it | **+0.0884** | 6 of 6 |
| it did not | **−0.0504** | 1 of 6 |
| difference | **+0.1388** | positive in 6 of 6 eras |

One bit of future information, whether the relationship still looked cointegrated once the test year
was folded in, is worth fourteen log points a year. That is larger than the edge that was being
measured. The gap is not partly look-ahead. The gap IS the look-ahead.

The section-4 dependence caveat applies here too: a cross-era t computes to 5.23 and overstates
precision for the same reason. The claims needing no aggregation are the strongest ones anyway:
the split is positive in every era separately, and in era 1, where formation and vintage windows
have identical length and power, the difference is +0.187 across 240 pairs in one year.

---

## 6. Does the ordering survive

Acceptance is one claim and the ranked list is another. The vintage's own Sharpe column ranked the
held-back year at a rank correlation of 0.150, monotone across deciles, a factor of 2.5 from bottom
to top. That column was computed on the full sample.

Recomputed before the split, across five eras, 30,000 pair-years:

| column | rho across eras | se | eras same sign |
|---|---|---|---|
| formation ADF t | −0.003 | 0.019 | 4 of 5 |
| formation Sharpe | −0.005 | 0.019 | 4 of 5 |
| formation return a year | +0.022 | 0.015 | 3 of 5 |
| trades a year | +0.027 | 0.014 | 4 of 5 |
| return correlation | +0.006 | 0.008 | 3 of 5 |
| half-life | −0.013 | 0.012 | 3 of 5 |
| spread deviation | −0.002 | 0.021 | 3 of 5 |

None of the seven reaches two standard errors, so none of them is a signal. But "inside its own
standard error" was too tidy a way to say that, and three of them are not: half-life sits at 1.15,
return a year at 1.55, and trades a year at 1.89. The two that lean furthest are both turnover
proxies rather than quality measures, and a rule that trades more collecting slightly more is not a
ranking.

What matters is which columns are dead. The formation Sharpe sits at 0.28 standard errors and the
ADF statistic at 0.14, and those are the two the product actually orders its list by. Quintiles of
formation Sharpe give held-back-year means of 0.0124, 0.0116, 0.0044, 0.0033, 0.0103: no order at
all, and the best quintile below the worst.

**The columns the screen ranks on carry nothing. The only two that lean at all are measures of how
often it trades.**

---

## 7. What this does not say

It does not say pairs trading does not work. It says this screen's outputs, on this universe, under
one fixed rule, carry no measurable forward information.

Specific limits. One trading rule was tested, entry at two sigma, exit at half, twenty-day stop,
twenty basis points a round trip; the product sweeps 108 cells, though the clean ranking test says
the formation-window backtest does not rank either. Six eras of 250 sessions is six observations,
and the earliest formation windows are 1,500 sessions rather than 2,750. Around 250 pairs pass in
each era, and they overlap in names, so the honest reading of the gap is that it is not detectably
positive rather than that it is reliably negative.

Findings measured independently of the acceptance label are unaffected: capacity, survivorship,
transaction costs, market neutrality, the borrow distribution.

---

## 8. Is the instrument sound

A negative result is worth what the instrument that produced it is worth. Before concluding that the
screen carries no forward information, the identical accept-and-trade code has to be shown finding
an edge that was put there deliberately.

Synthetic pairs: a stated fraction whose spread is a genuine Ornstein-Uhlenbeck process with a known
half-life, the rest two independent random walks sharing a market factor so both groups carry
similar correlation. Same split, same finite-sample critical value, same trading rule.

| half-life | accept, genuine | accept, independent | purity | gap |
|---|---|---|---|---|
| 15 sessions | 100.0% | 2.1% | 89.5% | **+0.120** |
| 25 sessions | 100.0% | 2.1% | 89.5% | **+0.092** |
| 40 sessions | 99.8% | 2.1% | 89.4% | **+0.081** |

The instrument works. Two things worth noting beyond that. The 2.1 per cent acceptance on
independent pairs is an independent confirmation of the joint-bootstrap null, which put the strict
rule's true size at 2.0 to 2.7 per cent, arrived at by a completely different route. And the gap it
recovers, +0.08 to +0.12, is the same size as the CONTAMINATED result on real data, +0.10 to +0.14.
The look-ahead was worth about as much as a real relationship would have been.

## 9. What size of effect is ruled out

Weaken the tie until the acceptance rate is realistic rather than certain, and vary how common
genuine cointegration is:

| prevalence | accept rate | purity of the accepted list | gap |
|---|---|---|---|
| 15% | 16.9% | 89.0% | +0.046 |
| 8% | 10.0% | 79.9% | +0.044 |
| 4% | 6.1% | 65.4% | +0.041 |
| 2% | 4.2% | 48.0% | +0.032 |
| 1% | 3.2% | 31.4% | +0.027 |
| 0.5% | 2.7% | 18.6% | +0.023 |

The real screen accepts 4.2 per cent of what it tests, which lines up with the 2 per cent row, and
the gap there would be +0.032. Measured, it is −0.021, with an era-to-era standard error of 0.013.

So this is not merely a failure to find something. Genuine cointegration present in even half a per
cent of tested pairs, at a strength that clears twenty basis points of cost, would have produced a
visibly positive gap. It did not. The bound is conditional on that strength: a real tie whose spread
is too small to cover costs would be invisible here, and would also be untradeable, which is the
same conclusion for a reader.

---

## 10. The one thing that does change the answer

The analysis began with the criticism that screening everything against everything manufactures
noise. Run the clean protocol separately inside each relatedness class, rejected arm matched on
correlation inside that same class, 6,000 pairs per class, five eras:

| class | accept rate | accepted arm, a year | gap vs matched rejects | t | gap positive |
|---|---|---|---|---|---|
| same industry | 5.99% | **+0.0209** | +0.0050 | 0.34 | 2 of 5 |
| same sector | 4.19% | **+0.0122** | −0.0011 | −0.09 | 3 of 5 |
| unrelated | 3.97% | **−0.0084** | −0.0253 | −3.07 | 0 of 5 |

Two readings, and they point the same way.

The prior matters and in the direction a desk would predict. Restricting to economically related
names moves the acceptance test from actively harmful to merely useless. Among unrelated pairs the
test selects pairs that go on to do WORSE than correlation-matched pairs that failed it, reliably,
in five years of five. That is what overfitting looks like from the inside: with no economic reason
for two names to be tied, passing a stationarity test means the spread happened to look tight, and
tight-by-accident is exactly what breaks next year.

But the value is in the names, not in the test. The rejected arms perform about the same across all
three classes, roughly +0.016 a year. What differs is the accepted arm. A reader who takes
same-industry pairs and trades their spread earns about two per cent a year on y-leg notional gross
of borrow at twenty basis points a round trip, and gets the identical result whether or not the pair
ever passed a cointegration test.

**The screen's contribution is the universe it looks at. The statistics on top of it contribute
nothing, and below the industry level they contribute less than nothing.**

---

## 11. Review, 2026-08-25

Sent for review to the product's design partner, a practising quantitative researcher. Four corrections came back, all of them right, all of
them checked against the stored frames rather than accepted on authority.

**The +13.9 is the asset, not a supporting detail.** Form pre-split, split the accepted group by
whether a full-sample run agrees, measure the gap. Pointed at any screen, that prices one bit of
forward information. It is a method with a worked example attached, and the worked example happens
to be our own product. Filing it as part of the finding undersells it.

**The +13.9 and the +5.5 are not off the same set, and the write-up put them side by side.**
Confirmed: +5.5 is accepted against correlation-matched REJECTED pairs, arms of 1,200 and 1,200.
+13.9 is measured INSIDE the in-era accepted group, arms of 115 and 125 falling to 29 and 224, and
every pair in both arms has already cleared formation. Saying the hindsight was worth more than
"the entire apparent edge" compares two different denominators. It is worth more than the edge that
was being measured, which keeps the point and closes the hole.

**The −0.021 was pushed further than it goes.** Six eras, five negative, and the dispersion was
never reported. Per era the gap runs −0.0036, −0.0249, −0.0048, −0.0093, −0.0197, −0.0133, and the
accepted arm itself swings from −0.031 to +0.039 year to year inside a single relatedness class. The
arms are roughly 250 overlapping pairs living through one year, so the era-to-era spread is not the
sampling error of the gap. This supports **accepted does not beat rejected**, which is the finding
either way. It does not support accepted losing to rejected, and nothing here needs it to.

**The clean positive control is the easy case.** One hundred per cent acceptance on an undegraded
Ornstein-Uhlenbeck spread establishes only that the test is not broken. The useful version grades
it: several half-lives and tie strengths, a drifting hedge ratio, a relationship that dies partway,
and the cell where acceptance falls to a half. That is a detection floor, and it converts the result
from "no ranking signal" into "no ranking signal above X", which is a claim rather than an absence.

Also raised, and being measured rather than argued: the two per cent on notional was quoted without
its turnover, and a return of that size at this horizon means little until the number of round trips
is on the table. If it is gross, or if it is industry-neutral residual reversion, then it is a
documented and crowded premium being collected rather than anything this screen discovered.

---

## 12. The graded control, and where the screen actually goes blind

The reviewer's fourth point was the valuable one. One hundred per cent acceptance on a clean Ornstein-Uhlenbeck
spread is the easy case and establishes only that the test is not broken. Graded across half-lives,
tie strengths and three degradations, 200 true pairs against 1,200 false in each cell:

**Constant tie.** Detection never degrades. Acceptance on genuinely cointegrated pairs holds at 100
per cent down to the faintest strength tested, and only slips to 89 to 94 per cent at a 60-session
half-life. Two thousand five hundred observations is a great deal of power. The detection floor for
a tie that never moves sits below anything worth trading, so that is not where the screen fails.

**Drifting hedge ratio.** This is where it fails. Sweeping the drift and recording the noise it
injects relative to the spread's own size:

| noise from drift, as a multiple of the spread | cells | acceptance, low to high | median |
|---|---|---|---|
| up to 0.7x | 14 | 100 to 100% | 100% |
| 0.7 to 1.2x | 4 | 92 to 100% | 99% |
| 1.2 to 2.0x | 6 | 55 to 99% | 82% |
| 2.0 to 3.2x | 6 | 29 to 77% | 48% |
| 3.2 to 5.0x | 6 | 14 to 37% | 26% |
| 5.0 to 8.0x | 4 | 7 to 19% | 11% |
| above 8x | 2 | 2 to 7% | 4% |

Detection is intact below about 1.2 times and the median halves in the two-to-three-times band. At
the top end, above eight times, the two cells give 2.0 and 6.5 per cent against a false rate running
1.5 to 3 per cent throughout, so one has reached the floor and the other is still three times above
it. It approaches the floor there rather than sitting on it.

The ratio is NOT a sufficient statistic and calling this a clean scaling law overstated it. Half-life
matters as much: at 1.6 times the contamination a 15-session half-life still accepts 98 per cent
while a 30-session one accepts 67. Taking each half-life separately, acceptance falls to a half at
3.6 times for a 15-session tie and 2.2 times for a 30-session one.

In drift terms that is roughly 20 to 25 per cent for a strong tie and 7 to 11 per cent for a faint
one, over eleven years, with the faster-reverting pairs tolerating more. Real companies move further
than that. A test that only sees relationships whose ratio holds is not seeing the interesting
cases.

**A relationship that dies.** The most important cell. A tie that reverts normally and then stops
for the last fifth of the formation window is:

| | |
|---|---|
| still accepted | **78 to 88%** at 15 and 30-session half-lives |
| gross return | +0.001 to +0.016 |
| net of costs | **negative in 13 of 15 cells** |
| gap over rejects | −0.022 to +0.016, centred on zero |
| ranking correlation | −0.072 to +0.051, centred on zero |

The test cannot tell a live relationship from a dead one. And that combination, accepted at a high
rate, earning nothing net, no gap, no ranking, is the real-data signature exactly.

So the screen is probably not finding relationships that never existed. It is finding relationships
that HAVE existed and no longer do, and it has no way to tell the difference, because a decade of
history outvotes the last two years.

---

## 13. The repair the diagnostics pointed at, tested

If the screen is finding dead relationships because a decade of history outvotes the last two years,
the obvious repair is to run acceptance on the recent part of the formation window instead. Three
rules on identical data, every decision made before the split, 6,000 pairs across five eras:

| rule | accept rate | accepted arm a year | gap vs matched rejects | se | t | gap positive |
|---|---|---|---|---|---|---|
| full window, as shipped | 4.18% | −0.0033 | **−0.0201** | 0.0052 | −3.87* | 0 of 5 |
| last 500 sessions only | 3.34% | +0.0017 | **−0.0034** | 0.0138 | −0.24 | 1 of 5 |
| accepted under both | 0.55% | +0.0088 | +0.0066 | 0.0390 | 0.17 | 1 of 2 |

The starred t carries the section-4 caveat: eras overlap, so cross-era t-statistics overstate
precision, and the per-era signs are the honest strength claim.

The diagnosis was right and the repair does what it was predicted to do. Testing on the recent
window removes the damage: the gap moves from −0.020 with a t of −3.87 to −0.003 with a t of −0.24,
and the accepted arm stops losing money. Stale acceptance was indeed the source of the harm.

It does not create an edge. It recovers to zero. The intersection rule accepts 0.55 per cent of
pairs, which is 21 to 45 names an era, and a standard error of 0.039 on a gap of 0.007 means that
row says nothing at all and should not be read as encouraging.

One wrinkle worth stating: era 5's formation window is 1,500 sessions, so its "last 500" is a third
of the window rather than a fifth, and its acceptance rate jumps to 6.3 per cent against 2.3 to 2.9
elsewhere. It is the one era where the recency gap is near zero rather than negative.

**Verdict.** The most promising repair the diagnostics pointed at takes the acceptance test from
harmful to neutral. Nothing found here takes it to useful. That is the third independent route to
the same place, after the clean walk-forward and the ranking test, and it is the point at which
continuing to tune the screen stops being research and starts being hope.

---

## 14. The gate is orientation-free. The number you trade is not.

Raised in review on 2026-08-25, and it is the sharpest structural criticism the work has had.

The both-orientations rule makes ACCEPTANCE symmetric, which was the right answer to the ordering
complaint. It does nothing for the hedge ratio. A reader still has to trade one number, and the two
one-sided fits do not agree. Across all 50,407 accepted pairs:

| the two OLS fits, one inverted, disagree by | |
|---|---|
| 25th percentile | 11.6% |
| median | **22.0%** |
| 75th percentile | 41.4% |
| 90th percentile | 73.7% |
| more than 10% | 79.7% of accepted pairs |

It behaves exactly as regression attenuation predicts. Median gap 33.2% where return correlation is
below 0.2, falling to 6.0% above 0.6. The weaker the relationship, the less determined the number.

**And the product does choose, on the worst possible basis.** The displayed columns come from
whichever orientation produced the more negative test statistic. Checked against the stored
per-orientation figures, that is 100 per cent consistent, and it flips 55.5 per cent of accepted
pairs relative to the raw ordering. So the hedge ratio a reader trades is selected on the strength
of the test that judged it. That is the same defect as letting an information criterion pick the lag
inside the gate: the data picks the estimator and the estimator then grades the data. It was written
down in a manifest note the whole time.

### Does the choice cost anything

Five estimators, each fitted on the formation window and frozen, judged on the held-back year.
Neutral alphabetical baseline, five eras, roughly 500 accepted pairs each.

| estimator | a year | vs neutral | paired t | better in |
|---|---|---|---|---|
| alphabetical | −0.00267 | | | |
| reversed, inverted | −0.00224 | +0.00043 | +0.06 | 3 of 5 |
| more negative t | −0.00209 | +0.00058 | +0.15 | 3 of 5 |
| total least squares | −0.00614 | −0.00347 | −0.62 | 2 of 5 |
| midpoint | −0.00403 | −0.00136 | −0.47 | 3 of 5 |

Nothing separates them. The number is undetermined by a fifth and it does not appear to matter which
end of that range gets traded, which is consistent with everything else here: choosing between five
ways of trading something that carries no forward edge.

### The most pointed demonstration of look-ahead in this document

Two attempts at the above were wrong first. The first ran on all tested pairs, where inverting a
near-zero slope explodes; the median gap came out at 198 per cent and the 90th at 5,711, which is a
fact about division. The second is the one worth keeping.

It used the vintage's own y_sym and x_sym as the neutral baseline. They are not neutral: they ARE
the more-negative-t orientation, chosen on the full 2,500 sessions including the year under test.

| arm | contaminated baseline | neutral baseline |
|---|---|---|
| reversed, inverted | −0.031, t −2.88 | +0.0004, t +0.06 |
| more negative t | −0.018, t −2.49 | +0.0006, t +0.15 |
| total least squares | −0.021, t −2.18 | −0.0035, t −0.62 |
| midpoint | −0.016, **t −3.36** | −0.0014, **t −0.47** |

Same test, same data, same code. One leaked input. It produced four significant-looking results
where there is nothing, including a t of −3.36. That is what this failure mode does, and it is worth
having a case where the before and the after sit next to each other.

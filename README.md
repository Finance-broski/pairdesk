# PairDesk

A pairs screener for US equities that publishes the test it failed.

Each build screens roughly 1,900 liquid US names, 1.7 million pairs, with
Engle-Granger both directions at finite-sample critical values, walk-forward hedge ratios,
a 108-cell parameter sweep, per-leg dollar volume with capital conversion, market-beta per spread, and a
provenance stamp on every number. The sample in this repository is a real weekly build.

## The part you should read first

Before selling anything, the screen was pointed at itself: acceptance relabelled inside each
test year so no figure could know the period it was judged on. The result, in
`SELECTION_LOOKAHEAD.md` (fourteen sections, audited, reproducible): the apparent edge was
selection look-ahead worth 13.9 points a year, accepted pairs do not beat correlation-matched
rejected ones, and no screening column ranks the year ahead. What survives is measured too:
economic relatedness clears every null we could build, capacity is real, and the accepted list
turns over about 3.3 percent per build while the standings move every session.

So this screen sells computation, freshness and correctness. Not forward information. The
methodology page inside the desk says exactly that, because a screen that shows only its
survivors is telling its reader the least useful true thing it knows.

## Files

| file | what |
|---|---|
| `desk_sample.html` | a full weekly build, self-contained, open it in a browser. No internet, no tracking, everything inside the file |
| `brief_sample.txt` | the one-page covering note a weekly refresh ships with |
| `SELECTION_LOOKAHEAD.md` / `.pdf` | the self-test: protocol, era tables, the audits, what died and what survived |
| `DECAY.md` | how fast the desk's information goes stale, measured on our own builds |

The writeup names the internal analysis scripts it ran on; those are not included here. The
protocol descriptions are complete enough to reimplement independently, and that is the
reproducibility on offer: the method, not our code.

One omission, stated rather than discovered: the free sample carries every statistic, the
spread and standing series, the parameter surfaces and the full register, but not the per-leg
price series (the two views drawn from them show their empty state). Prices are the exchanges'
and the vendors' to sell; everything computed FROM them here is ours to give away.

The desk snapshot ages by design; freshness is the whole game in screening, which the 3.3
percent turnover figure quantifies. The sample is complete so any judgment about the approach
can be made on real output.

Built by Finance-broski. Documents (README, writeup, PDF) are CC-BY 4.0: read, share, build on
them with attribution. The sample files (desk, brief) are free for personal research use and
not for redistribution as data.

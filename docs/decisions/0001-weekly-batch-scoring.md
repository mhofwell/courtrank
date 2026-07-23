# ADR 0001 — Weekly batch scoring replaces sequential scoring

| | |
|---|---|
| Status | Accepted · deployed in production |
| Date decided | 2026-06-21 |
| Date recorded | 2026-07-23 |
| Implementation | `apply_week` weekly accumulator |
| Evidence | [`../experiments/BASELINE_BATCH.md`](../experiments/BASELINE_BATCH.md) |
| Supersedes | Sequential per-game updates as described in White Paper ≤ v1.1 §2.2 and Appendix C |

## Context

The original simulator applied games in input order: score game 1, update ratings
and games-played, use those new values for game 2, and so on. Results therefore
depended on spreadsheet row order — an arbitrary artifact, since a pickleball
session is conceptually simultaneous. Later-listed games faced changed ratings and
smaller K-factors purely because of where they sat in the file.

## Decision

At session start the engine freezes every player's rating, games-played count,
contribution share, K-factor, and every opponent/partner rating used in the
performance calculation. Each game is evaluated against that frozen snapshot.
Deltas accumulate and are applied once:

```
R_after(i) = R_start(i) + sum over session games of dR(i, g)
```

Games-played increments only after the batch completes. Shuffling the game list
produces identical output.

## Measured impact

The behavioral change was statistically negligible:

| Metric | Sequential | Batch |
|---|---|---|
| Baseline Week 1 rho | 0.837 | 0.836 |
| Baseline Week 8 rho | 0.905 | 0.905 |
| Within one court, Week 8 | 92.2% | 92.1% |
| Severe mis-seed, Week 8 court | 2.78 | 2.70 |
| Rating drift | +0.1 | +0.1 |

## What we gained

Input-order independence, reproducibility, easier auditing, and a truthful model
of a single weekly observation period — at no measurable accuracy cost.

## White-paper implications

- §2.2 must state: all games in a session are evaluated against week-start state;
  ratings do not update between games within a session.
- The multi-game worked example in Appendix C must be regenerated with the same
  starting ratings and K for every game in the week. The single-game example
  remains valid if recalculated against current source.

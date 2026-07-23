# Sweep-pull blend selection (perf_blend 0.6 → 0.1)

_Configuration fingerprint: weekly batch scoring; rank clamp disabled;
`perf_window_games = 12`; blend sweep from 0.0 through 0.6. The 0.1 column is
the production baseline introduced in commit `f536c62`._

Why the league dampens the sweep performance-pull from the engine default 0.6 to
0.1. Evidence from `validate_pull_blend.py` (150 seasons/scenario, on the
production overlay: rank_clamp off, perf_window_games=12).

## Motivation

The pull fires on a sweep (all-win / all-loss night) to fast-correct a misseeded
shadow rating. At 0.6 it over-reacted to single-night noise in *both* directions:
- Case J vaulted toward #1 off two sweeps without playing a top-court opponent.
- Case S's 0-7 week 1 was 61% sweep pull (−203 of −331); the 6-1 week 2
  then couldn't dig her out, so a strong week felt unrewarded.

A rec-league sweep is noisy (draw/partner luck, off nights), so it should nudge,
not dominate.

## Results (150 seasons each)

```
baseline health            wk1 rho  wk4 rho  wk8 rho  exact wk8  +-1 wk8
 blend 0.0 (eliminate)       0.857    0.887    0.913     54.7%    92.7%
 blend 0.1  (chosen)         0.856    0.889    0.913     54.8%    93.0%
 blend 0.2                   0.854    0.886    0.911     54.9%    92.9%
 blend 0.3                   0.851    0.882    0.910     54.1%    92.2%
 blend 0.6 (engine default)  0.836    0.872    0.903     54.3%    92.2%

sandbagger (true top-3 seeded last, ~700pt) — avg court by week (1=top)
 blend 0.0   6.0 6.0 5.9 5.8 5.5 5.3 5.1 4.9
 blend 0.1   6.0 6.0 5.8 5.5 5.2 5.1 4.8 4.6
 blend 0.2   6.0 5.9 5.6 5.3 5.0 4.8 4.5 4.3
 blend 0.3   6.0 5.7 5.4 5.1 4.8 4.6 4.4 4.1
 blend 0.6   6.0 5.3 4.7 4.4 4.1 3.9 3.7 3.5

mid-season joiner (seeded 300pt low, joins wk3) — avg court from wk3
 blend 0.0   3.9 3.4 3.2 3.0 2.9 2.7
 blend 0.1   3.9 3.4 3.1 2.8 2.8 2.5
 blend 0.2   3.9 3.4 3.1 2.8 2.7 2.5
 blend 0.3   3.9 3.4 3.1 2.7 2.6 2.5
 blend 0.6   3.9 3.1 2.9 2.5 2.4 2.2
```

## Why 0.1

- **Baseline accuracy is unharmed** — lowering the pull actually nudges wk8 ρ up
  (0.903 → 0.913); the pull's single-night noise was a small drag on typical play.
- **On our real two weeks, 0.1 == eliminate** for the cases that drove this:
  Case S reaches Courts 3-4 (#28), while Case J lands top court but uncrowned
  (#6). At 0.2 Case S
  falls back to Courts 5-6, so ≤0.1 is required to satisfy the fairness goal.
- **0.1 keeps a misseed safety net that 0.0 discards.** The realistic 300pt joiner
  still converges (2.5 vs 2.7 court by wk8); even the extreme sandbagger does
  better (4.6 vs 4.9). Same one-line config either way (the pull code stays), so
  0.1 is Pareto-better than eliminating.
- **Extreme misseeds stay organizer-fixable** by manual reseed (as done for Moe
  Pazurik), backstopping the slow automated correction at low blend.

Trade-off accepted: vs eliminate, Case S sits at #28 (last spot in Courts 3-4) rather
than a roomier #22 — the price of keeping the safety net.

# League-config validation (production overlay)

_Configuration fingerprint: clamp disabled and 12-game pull taper at
`perf_blend = 0.6`. Predates the final 0.1 blend and is not the production
baseline. Retained as evidence for ADR 0004._

`BASELINE_BATCH.md` validated the generic engine (rank clamp 6, untapered pull).
Production runs a deliberate overlay: **rank clamp disabled** (`rank_clamp=10^6`,
ladder strictly points-ordered) and, as of 2026-07-17, a **12-game shadow-pull
window** (`perf_window_games=12`: the pull's ±400 cap tapers linearly to zero
across a player's first 12 games; pipeline additionally drops `is_shadow` at 12
games — mathematically redundant with the taper, kept as belt-and-braces).

This file captures `compare_league_config.py` (150 seasons/scenario, seeds
matched to BASELINE_BATCH): A = validated defaults, B = league before the
window, C = league as deployed. Row A reproduces BASELINE_BATCH.md exactly,
so B/C deltas are paired and trustworthy.

Sim players get ~4 games/week, so the 12-game window spans sim-weeks 1-3
(≈2 weeks at the real league's 6-7 games/week).

## Headline metrics

```
baseline                     wk1 rho  wk4 rho  wk8 rho  exact wk8  +-1 wk8  max up
A defaults (validated)        0.836    0.869    0.905      53.7%    92.1%       6
B league pre-change           0.836    0.869    0.902      53.5%    92.0%      25
C league current              0.836    0.872    0.903      54.3%    92.2%      25

12% absences
A defaults (validated)        0.842    0.869    0.899      54.1%    92.5%       6
B league pre-change           0.842    0.868    0.896      53.8%    92.0%      25
C league current              0.842    0.875    0.902      54.1%    92.6%      25

absences + wk3 joiners
A defaults (validated)        0.840    0.845    0.885      51.0%    90.0%       6
B league pre-change           0.840    0.844    0.887      52.0%    90.3%      25
C league current              0.840    0.849    0.888      51.6%    90.4%      23

sandbagger (top-3 seeded last)
A defaults (validated)        0.739    0.827    0.879      50.4%    89.6%       6
B league pre-change           0.739    0.823    0.877      51.0%    89.7%      24
C league current              0.739    0.812    0.859      49.2%    89.2%      24
```

## Misseed trajectories (avg court by week; 1 = top)

```
sandbagger (true top-3, seeded dead last — ~700-pt misseed, shadow)
A defaults        [6.   5.57 5.06 4.46 3.79 3.31 2.95 2.70]
B league pre      [6.   5.29 4.49 3.94 3.36 3.07 2.83 2.67]
C league current  [6.   5.29 4.71 4.40 4.09 3.88 3.73 3.54]

wk-3 joiner seeded 300 pts low (from wk3)
A defaults        [3.86 3.55 3.14 2.67 2.28 2.23]
B league pre      [3.86 3.11 2.93 2.40 2.29 2.14]
C league current  [3.86 3.05 2.89 2.50 2.38 2.22]
```

Rating drift per player per season (baseline): A +0.1, B −0.4, C −0.3.

## Verdict

- **Baseline / absences / joiners: the deployed config is equivalent to the
  validated engine** — wk8 ρ within ±0.003, court accuracy within ±0.6 pp,
  drift negligible. The realistic misseed (300-pt joiner) converges to the same
  place (wk8 court 2.22 vs 2.23) and *faster* early (no clamp).
- **The one measured cost: extreme sandbaggers converge slower.** A ~700-pt
  shadow misseed reaches court ~3.5 by wk8 instead of ~2.7, and that scenario's
  wk8 ρ drops 0.877 → 0.859. This is the direct price of bounding lifetime pull
  correction to ~322 pts: corrections beyond that must be earned through the
  courts at Elo speed. Deliberate trade (2026-07-17): the league prefers
  outliers to prove it at each tier over teleporting past players they've never
  faced. Egregious misseeds remain fixable by organizer re-seed.
- `max up-move 23-25` reflects the clamp removal (pre-existing, deliberate),
  not the pull window — it is identical in B and C.

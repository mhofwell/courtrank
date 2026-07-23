# ADR 0005 — Performance-pull blend reduced from 0.6 to 0.1

| | |
|---|---|
| Status | Accepted · deployed in production |
| Date decided | 2026-07-17 |
| Date recorded | 2026-07-23 |
| Implementation | production `CONFIG` sets `perf_blend = 0.1` |
| Configuration introduced | commit `f536c62` |
| Evidence | [`../experiments/BASELINE_PULL_BLEND.md`](../experiments/BASELINE_PULL_BLEND.md) (2,250 simulated seasons) |
| Supersedes | `lambda(g) = 0.6 * exp(−g/16)`; a clip-saturating debut sweep could move ±240 points |

## Context

At blend 0.6, the pull overreacted to single-session noise. One player's 0–7
Week 1 was dominated by the sweep correction (−321.5 total); another player's
Week 2 sweep vaulted them to #2 before they had faced top-court opposition. A
rec-league sweep still reflects partner draw, opponent draw, form, and a small
sample.

## Decision

Production uses `lambda(g) = 0.1 * exp(−g/16) * m(a)`, under the ADR 0004 taper
and eligibility rules. The pull's role changed: **a modest mis-seed nudge, not an
alternate rating engine.** Maximum pull by prior games (no absence multiplier):

| Games entering session | Clip | Blend | Max pull |
|---|---|---|---|
| 0 | 400.0 | 0.1000 | 40.00 |
| 4 | 266.7 | 0.0779 | 20.77 |
| 6 | 200.0 | 0.0687 | 13.75 |
| 7 | 166.7 | 0.0646 | 10.76 |
| 8 | 133.3 | 0.0607 | 8.09 |
| 11 | 33.3 | 0.0503 | 1.68 |
| 12 | 0 | ineligible | 0 |

At 6–7 games per week, the maximum ordinary two-session pull is ~51–54 points
(both sessions qualifying, clip-saturating sweeps) — versus the previous
several-hundred-point fast-track.

## Measured impact

Blends 0.0, 0.1, 0.2, 0.3, 0.6 tested across 2,250 simulated seasons. Relative
to the tapered 0.6 overlay:

| Measure | Blend 0.6 | Blend 0.1 |
|---|---|---|
| Baseline Week 8 rho | 0.903 | 0.913 |
| Exact court, Week 8 | 54.3% | 54.8% |
| Within one court, Week 8 | 92.2% | 93.0% |
| Severe mis-seed, Week 8 court | 3.54 | 4.61 |
| 300-point-low joiner, Week 8 court | 2.22 | 2.53 |

Suppressing single-night noise **improved** ordinary ranking accuracy. A
realistic 300-point mis-seed still corrects meaningfully. A ~700-point extreme
mis-seed no longer auto-corrects to the top courts within eight weeks — manual
organizer reseeding is now the intended remedy for an obviously absurd seed.

## Real-league impact (other data corrections held constant)

**Case S** — Week 1: 1380.0 → 1222.8 (−157.2 = −124.2931 ordinary Elo +
−32.8657 sweep pull), rank #31 (was −321.5 to #33 at 0.6). Week 2 (6–1, not a
sweep, ordinary Elo only): +94.4 → 1317.3, rank #28 (was #39 at 0.6). The lower
Week 2 gain is not a penalty: the player started Week 2 much higher, so more was
expected, while the resulting standing was eleven places better.

**Case J** — Week 2 sweep (7–0): +146.1 → 1533.8, rank #6 (was +214.8 →
1602.6, #2). Still rewarded strongly, but not crowned before accumulating
evidence against the top field.

**Case V** — Week 1 (0–7): −208.8 → 1101.2 (was −408.8 → 901.2). The 200-point
difference is exactly the reduction in a clip-saturated debut pull:
`(0.6 − 0.1) * 400 = 200`.

## Fresh validation of the fully deployed configuration

150 seasons per scenario at `rank_clamp` disabled, `perf_window_games = 12`,
`perf_blend = 0.1`:

| Scenario | W1 rho | W4 rho | W8 rho | Exact court W8 | ±1 court W8 |
|---|---|---|---|---|---|
| Baseline | 0.856 | 0.889 | 0.913 | 54.8% | 93.0% |
| 12% absences | 0.851 | 0.882 | 0.905 | 55.0% | 92.6% |
| Absences + Week 3 joiners | 0.851 | 0.857 | 0.895 | 52.3% | 90.7% |
| Severe top-3 player seeded last | 0.726 | 0.786 | 0.842 | 50.1% | 88.8% |

Severe mis-seed weekly court trajectory (court 1 best):
`[6.00, 5.96, 5.77, 5.53, 5.24, 5.09, 4.79, 4.61]`
300-point-low Week 3 joiner: `[3.91, 3.42, 3.13, 2.82, 2.80, 2.53]`

## White-paper implications

Replace §4's simulation matrix and the §4.1 severe-mis-seed chart/narrative with
the tables above. Frame conservatism toward extreme seeds as an intentional
safety choice: normal seed error self-corrects; extreme seed error proceeds
conservatively; organizer reseeding handles obvious administrative mistakes; the
algorithm does not let one or two sessions overpower all prior placement
evidence.

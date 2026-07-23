# CourtRank White Paper v2.0 revision briefing

_Prepared July 2026. Public edition; live-league case studies are anonymized._

## Why this is a major version

White Paper v1.1 would mislead an organizer in two actionable areas:

1. It implies that a severely mis-seeded player will automatically reach the
   top courts during an eight-week season. Under the deployed 0.1 performance
   blend, a roughly 700-point mis-seed reaches only about court 4.6 by Week 8.
   Manual organizer reseeding is now the intended remedy for an obviously
   absurd seed.
2. It does not describe the deployed weekly batch semantics. A multi-game
   session is evaluated against one frozen week-start state, so its arithmetic
   differs from the sequential worked example in v1.1.

The product promise and a worked example both change. The next paper is
therefore v2.0, not v1.2.

## What remains unchanged

The foundation is still a margin-aware, contribution-weighted doubles Elo:

- linear DUPR seeding;
- partner-average team ratings;
- Elo win expectation;
- a 50/50 blend of win/loss and score margin;
- a bounded 35%–65% within-team contribution allocation;
- adaptive K decaying from 56 toward 32;
- an absence uncertainty multiplier capped at 1.5×;
- a winner floor preventing a winner from receiving a negative game delta.

## Current scoring model

Seed:

```text
R0 = 1500 + 200 * (DUPR - 3.5)
```

Team expectation:

```text
RA = mean(Ri, Rj)
RB = mean(Rk, Rl)
EA = 1 / (1 + 10^((RB - RA) / 400))
```

Margin-aware result:

```text
M = 0.5 + 0.5 * (points_for - points_against) / 11
S = 0.5 * win_indicator + 0.5 * M
```

Adaptive update:

```text
K(g) = 32 + 24 * exp(-g / 10)
```

The stronger-rated partner receives a slightly larger magnitude, clipped so
neither partner receives less than 35% or more than 65% of the team swing.
This is a bounded heuristic; durable attribution comes from rotating partners.

## Decision 1: weekly batch scoring

The original simulator updated ratings after each game. Later games therefore
used different ratings and K-factors solely because of spreadsheet order.

Production freezes ratings, games played, contribution shares, K-factors, and
performance-observation ratings at session start. Every game is scored against
that snapshot, deltas are accumulated, and the state is updated once:

```text
R_after(i) = R_start(i) + sum(dR(i, game))
```

Shuffling input games now produces identical output. Revalidation found no
meaningful accuracy loss: baseline Week 8 rho remained 0.905 and within-one-
court accuracy changed from 92.2% to 92.1%.

## Decision 2: strict rating sort

The former six-position movement clamp allowed rank and rating to contradict
each other. Production disables the clamp and ranks by descending rating.

There is no formal one-week movement guarantee. Stability is empirical:
current simulations found at least 99.1%–99.4% of player-weeks moved no more
than six rank positions, while real mis-seeds can move farther.

## Decision 3: no rating floor

A temporary 1200-point floor became a sticky attractor because the floored
value re-entered the next session. It created artificial ties and erased
evidence at the bottom of the ladder.

Ratings now persist exactly as computed, including values below 1200. They are
internal ranking state, not displayed as an inferred public DUPR.

## Decision 4: 12-game shadow-pull window

The sweep-triggered pull is now an early-seeding correction rather than a
season-long uncertainty mechanism.

For a shadow-rated player entering a session with fewer than 12 games, the pull
can fire only after at least three games and only when every result is a win or
every result is a loss.

Its clip tapers linearly:

```text
c(g) = 400 * max(0, 1 - g / 12)
```

Eligibility ends at 12 prior games. At the real league's six to seven games per
session, this is approximately two sessions.

## Decision 5: performance blend 0.1

The old 0.6 blend could move a clip-saturating debut player by 240 non-Elo
points. Production uses:

```text
lambda(g) = 0.1 * exp(-g / 16) * absence_multiplier
```

At debut the maximum ordinary pull is 40 points. At six prior games it is about
13.75 points, and at twelve it is zero.

The role is now a modest mis-seed nudge, not an alternate rating engine.

## Measured impact of the final blend

| Measure | Tapered blend 0.6 | Deployed blend 0.1 |
|---|---:|---:|
| Baseline Week 8 rho | 0.903 | 0.913 |
| Exact court, Week 8 | 54.3% | 54.8% |
| Within one court, Week 8 | 92.2% | 93.0% |
| Severe mis-seed, Week 8 court | 3.54 | 4.61 |
| 300-point-low joiner, Week 8 court | 2.22 | 2.53 |

Reducing the pull improved ordinary-season accuracy by suppressing
single-session noise. The cost is deliberately slower automatic repair of
extreme mis-seeds.

## Anonymized production cases

- **Case S:** a 0–7 Week 1 changed from −321.5 at blend 0.6 to −157.2 at blend
  0.1. The deployed result was −124.2931 ordinary Elo plus −32.8657 pull.
  A following 6–1 session was not a sweep and earned +94.4 ordinary Elo.
- **Case J:** a 7–0 Week 2 changed from +214.8 and rank #2 to +146.1 and rank
  #6. The sweep remained strongly rewarded without prematurely crowning the
  player.
- **Case V:** a 0–7 Week 1 changed from −408.8 to −208.8. The exact 200-point
  difference is `(0.6 - 0.1) * 400`.

## Fully deployed simulation matrix

Each row is 150 simulated eight-week seasons at batch scoring, no rank clamp,
a 12-game taper, and blend 0.1.

| Scenario | W1 rho | W4 rho | W8 rho | Exact W8 | Within one court W8 |
|---|---:|---:|---:|---:|---:|
| Baseline | 0.856 | 0.889 | 0.913 | 54.8% | 93.0% |
| 12% absences | 0.851 | 0.882 | 0.905 | 55.0% | 92.6% |
| Absences + Week 3 joiners | 0.851 | 0.857 | 0.895 | 52.3% | 90.7% |
| Top-three player seeded last | 0.726 | 0.786 | 0.842 | 50.1% | 88.8% |

Severe mis-seed court by week, where court 1 is best:

```text
[6.00, 5.96, 5.77, 5.53, 5.24, 5.09, 4.79, 4.61]
```

Week 3 joiner seeded 300 points low:

```text
[3.91, 3.42, 3.13, 2.82, 2.80, 2.53]
```

## Required v2.0 paper changes

1. State the weekly batch rule in the algorithm specification.
2. Qualify exact zero-sum language: unequal K, absence multipliers, and the
   winner floor create small deviations. Baseline drift is approximately
   +0.045 points per player per eight-week season.
3. Replace “win and rise, lose and fall” with expectation-and-margin language;
   a narrow underdog loss can outperform expectation.
4. Clarify that verified and shadow players share ordinary adaptive K; only
   shadows receive the temporary sweep correction.
5. State that an absentee's rating freezes but ordinal position can change.
6. Replace the simulation table and severe-mis-seed figure with the deployed
   results above.
7. Reframe severe mis-seeds as organizer-correctable administrative errors.
8. Replace the parameter appendix with the table in `docs/CURRENT.md`.
9. Regenerate the multi-game worked example using frozen week-start state.
10. Include a version-history page and production configuration fingerprint.

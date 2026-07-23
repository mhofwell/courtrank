# ADR 0004 — Shadow-rating pull tapers linearly and expires at 12 games

| | |
|---|---|
| Status | Accepted · deployed in production |
| Date decided | 2026-07-17 |
| Date recorded | 2026-07-23 |
| Implementation | performance-pull taper in `apply_week`; 12-game eligibility policy in production configuration |
| Evidence | [`../experiments/BASELINE_LEAGUE.md`](../experiments/BASELINE_LEAGUE.md) (predates the final 0.1 blend) |
| Supersedes | Fixed ±400 pull clip with eligibility persisting ~54 games |

## Context

The sweep-triggered performance pull was originally a long-running uncertainty
mechanism: its ±400 clip never shrank, and eligibility persisted until the
exponential blend became negligible (~54 games). That let a single-night sweep
keep exerting non-Elo corrections deep into a season.

## Decision

The pull is now explicitly an **early-seeding correction**, not a running
uncertainty mechanism:

- Clip shrinks linearly: `c(g) = 400 * max(0, 1 − g/12)`.
- Eligibility ends outright at `g >= 12` prior games.
- Trigger conditions (unchanged in kind, now stated exactly): shadow-rated,
  fewer than 12 prior games, at least three games in the session, all wins or
  all losses.

**Why 12:** the real league plays ~6–7 games per session, so 12 games ≈ two full
sessions. Policy interpretation: *uncertainty belongs to the seed, not
permanently to the player.* After roughly two sessions, ordinary results have
supplied enough evidence that the special seeding mechanism should disappear.

## Measured impact (evaluated at the then-current 0.6 blend)

Ordinary scenarios essentially unchanged: baseline Week 8 rho within 0.003 of
the generic engine; court accuracy within 0.6 percentage points; a realistic
300-point-low joiner reached approximately the same destination.

Primary effect was on extreme mis-seeds: untapered severe sandbagger reached
~court 2.7 by Week 8; tapered, ~court 3.5. Deliberate trade: corrections beyond
the early safety window must be earned through normal Elo results or handled by
an organizer reseed.

## White-paper implications

- §2.4 must add `c(g)`, the 12-game eligibility cutoff, and the exact trigger
  conditions.
- Qualify the "roughly six-percent sweep fluke" claim by game count (balanced
  games): 4-game sweep either way 12.5%, 5 games 6.25%, 6 games 3.125%,
  7 games 1.5625%.
- §5: replace "by week three the seeds have largely washed out" with: the
  special shadow-seed correction ends after approximately two real-league
  sessions, while ordinary Elo continues accumulating evidence all season.

# White paper — version history and policy

## Versioning policy

- **Major (X.0):** the algorithm or its promises change such that a reader of
  the previous version could act on a claim that is no longer true.
- **Minor (X.Y):** the claims survive, but calibration, evidence, or supporting
  material changes.
- **Errata:** wording, typography, and layout corrections.

Every release must include a production-configuration box and the date and
commit fingerprint to which it corresponds.

## History

### v2.0 — July 23, 2026

Major revision based on the production decisions recorded in ADRs 0001–0005:

1. Weekly batch scoring against week-start state.
2. Strict rating sorting with no movement clamp.
3. No 1200-point rating floor.
4. A shadow pull that tapers and expires at 12 games.
5. Performance-pull blend reduced from 0.6 to 0.1.

The severe-mis-seed promise changes materially: an approximately 700-point
mis-seed reaches about court 4.6 by Week 8 under the deployed configuration,
rather than automatically reaching the top courts. Obvious administrative
mis-seeds are now expected to be corrected by an organizer.

The v2.0 paper also qualifies zero-sum language, describes batch computation,
replaces the simulation matrix, updates the parameter appendix, and regenerates
the multi-game worked example.

### v1.1 — July 2026

Removed the six-position rank clamp and replaced it with a plain rating sort.
The paper added the full numbered mathematics, updated the simulation evidence
to describe naturally bounded movement without a formal cap, and expanded the
notation and worked-example appendices. It retained the original 0.6
performance blend and did not yet describe the 12-game taper.

### v1.0 — June 2026

Initial publication: margin-aware, contribution-weighted doubles Elo with
adaptive K, absence handling, winner floor, sequential within-session scoring,
a six-position weekly rank clamp, and a 0.6 performance pull.

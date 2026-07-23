# ADR 0002 — Rank movement clamp removed; rank is a strict rating sort

| | |
|---|---|
| Status | Accepted · deployed in production |
| Date decided | 2026-07-10 |
| Date recorded | 2026-07-23 |
| Implementation | production configuration sets `rank_clamp = 10**6` |
| Configuration fingerprint | production commit `5a00c2e` |
| Supersedes | The six-position/one-court weekly movement clamp in the generic engine defaults |

## Context

The generic engine limited weekly movement to six ordinal positions
(`|rank_new − rank_old| <= 6`), intended to bound movement to roughly one court.
In practice the clamp let rating and rank contradict each other: a lower-rated
player could sit above a higher-rated one, "points rank" and "official rank"
diverged, extra analytics were needed to explain who was owed positions, and the
guardrail delayed exactly the players whose seeds were most clearly wrong.

## Decision

Ranking is a plain descending sort of rating points. The ladder and the points
always agree. No formal cap on one-week movement exists.

## Evidence

- In real Week 1 data, 32 of 34 players naturally stayed within roughly one court
  group with no clamp. The two larger moves were genuine mis-seed corrections;
  one newcomer rose 20 positions under strict points ranking.
- Fresh simulation at the current production configuration (see ADR 0005): at
  least 99.1%–99.4% of player-weeks across four scenarios moved no more than six
  rank positions; moves exceeding twelve positions occurred in at most 0.007% of
  simulated player-weeks.
- Maximum observed upward move: ~25 positions under the old 0.6 untapered blend,
  falling to 12–13 positions under the deployed 0.1 blend. Empirical, not a
  guarantee.

## Trade-off accepted

Rare large single-week moves are possible. Stability is *empirical*, created by
bounded rating updates — not enforced by rank manipulation.

## White-paper implications

- Keep the "plain sort, no movement cap" description; add the empirical movement
  evidence above and the no-formal-cap caveat.
- Retire "Ones to Watch" / `spots_owed` style analytics — no longer needed.
- New players enter directly at the position implied by their seed.

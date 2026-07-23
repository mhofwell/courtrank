# ADR 0003 — The 1200-point rating floor removed

| | |
|---|---|
| Status | Accepted · deployed in production |
| Date decided | 2026-07-10 |
| Date recorded | 2026-07-23 |
| Implementation | stored rating = engine rating, unclamped |
| Supersedes | The brief pipeline clamp `R_stored = max(1200, R_engine)` |

## Context

The pipeline briefly clamped stored ratings to 1200. The floor was not cosmetic:
the floored value re-entered the next week's calculation, creating a **sticky
attractor**. Consequences: multiple struggling players piling up at exactly 1200,
artificial ties, loss of information about relative performance at the bottom,
distorted future expectations, and rank order once again diverging from the
engine's actual evidence.

## Decision

The exact computed rating persists, both for new-player seeding below DUPR 2.0
and for post-session ratings below 1200. Ratings are unbounded internal state.
Public-facing adjusted DUPR values are not inferred or displayed, so there is no
reason to force the internal scale above an arbitrary minimum.

## Evidence

In the current Week 1 production run, a winless player finishes at 1101.2
rather than being lifted to 1200.

**Historical-number caveat:** older design documents cite outputs of 1049.4 and
901.2. Those values came from the former 0.6 performance blend (see ADR 0005).
They remain useful for explaining why a floor is structurally harmful, but are
not current league outputs.

## White-paper implications

State that ratings are unbounded internal state and that no minimum is imposed at
seeding or after sessions.

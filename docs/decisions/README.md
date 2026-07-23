# Decision records

This folder is the append-only history of *why* CourtRank works the way it does.

## Convention

- One file per decision, numbered in the order recorded: `NNNN-short-slug.md`.
- Records are **never edited to stay current**. If a decision is reversed or revised,
  write a new record and mark the old one `Superseded by ADR NNNN` in its status line.
- Each record captures: the context and problem observed, the decision, the measured
  impact, the trade-off accepted, and pointers to code and evidence files.
- "How it works today" lives in `docs/CURRENT.md` and the white paper — not here.
  These records are history and cannot go stale.

## Why this exists

`docs/how-the-ladder-works.md` drifted out of date because it mixed *current state*
with *rationale*. When the production configuration changed (see ADRs 0001–0005),
the rationale survived but the stated parameters silently became wrong. Splitting
the two document types prevents a repeat.

## Index

| ADR | Decision | Status |
|-----|----------|--------|
| 0001 | Weekly batch scoring replaces sequential scoring | Accepted, deployed |
| 0002 | Rank movement clamp removed | Accepted, deployed |
| 0003 | 1200-point rating floor removed | Accepted, deployed |
| 0004 | Shadow pull tapers and expires at 12 games | Accepted, deployed |
| 0005 | Performance-pull blend reduced from 0.6 to 0.1 | Accepted, deployed |

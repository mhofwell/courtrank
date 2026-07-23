# CourtRank — current production specification

_Last verified: 2026-07-23 · target configuration for White Paper v2.0_

This document describes the deployed CourtRank behavior. The implementation
remains in a private application repository; this public repository contains the
papers, decision history, revision notes, and reproducible result snapshots.

## Configuration fingerprint

| Field | Value |
|---|---|
| Documentation snapshot | 2026-07-23 |
| Production repository commit | `5a00c2e` |
| Final 0.1 configuration introduced | `f536c62` |
| White paper | v2.0 in progress |

If an older paper or experiment report disagrees with this document, this
document and the later decision record take precedence.

## Production parameters

| Parameter | Current production value |
|---|---|
| Seed anchor | 1500 at DUPR 3.5 |
| Seed scale | 200 points per DUPR |
| K (new) | 56 |
| K (steady) | 32 |
| K decay | 10 games |
| Win/margin weight | 0.5 / 0.5 |
| Contribution share | clipped to 35%–65% |
| Winner floor | enabled |
| Absence boost | +15% per missed week, max 1.5× |
| Shadow-pull blend | 0.1 |
| Shadow-pull decay | 16 games |
| Dead zone | 150 points |
| Initial pull clip | ±400 |
| Pull taper | linear to zero over 12 games |
| Pull eligibility | shadow, fewer than 12 prior games, at least three games, all-win/all-loss sweep |
| Rank clamp | disabled |
| Rating floor | none |

## Current model

CourtRank is a weekly-batch, margin-aware doubles Elo system. It evaluates every
game against the session's starting ratings, distributes each team adjustment
through a bounded contribution heuristic, and adapts update size as players
accumulate evidence. Shadow-rated players receive a small sweep-triggered seed
correction during their first twelve games; after that, every player advances
through ordinary results. Rankings are a direct sort of unconstrained rating
points, so rank always agrees with the evidence.

The design favors accurate ordinary-season ordering and transparent movement
over aggressive automatic repair of extreme administrative mis-seeds.

## Authority and history

- [Decision records](decisions/README.md) explain why the current behavior was
  chosen.
- [Experiment reports](experiments/) preserve the configuration and results
  used for each decision.
- [White-paper version history](whitepaper-versions.md) records which public
  claims belonged to each version.
- [White Paper v1.0](../papers/courtrank-whitepaper-v1.0.pdf) and
  [v1.1](../papers/courtrank-whitepaper-v1.1.pdf) are retained as historical
  publications and are not current specifications.

## Reporting conventions

- Improvement is measured from a player's entry seed, including for mid-season
  joiners.
- Weekly movement is pre-session versus post-session; season movement is entry
  versus current.
- An absentee's rating is frozen, but their ordinal position is not: other
  players can pass them.

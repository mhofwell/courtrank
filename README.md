# CourtRank

Public documentation for CourtRank, a transparent ranking system for
rotating-partner doubles pickleball ladders.

This repository intentionally contains the publication layer only:

- historical and current white papers;
- the current production specification;
- append-only architecture decision records;
- algorithm revision briefings;
- configuration-stamped simulation reports.

The production engine, ingestion pipeline, league data, and administrative
application remain private.

## Start here

- [Current production specification](docs/CURRENT.md)
- [Decision history](docs/decisions/README.md)
- [Changelog and versioning policy](CHANGELOG.md)
- [White Paper v1.0](papers/courtrank-whitepaper-v1.0.pdf)
- [White Paper v1.1](papers/courtrank-whitepaper-v1.1.pdf)
- [White Paper v2.0 — current](papers/courtrank-whitepaper-v2.0.pdf)
- [v2.0 revision briefing](docs/revisions/2026-07-whitepaper-v2-briefing.md)

White Paper v2.0 is the current publication.

## Scope

These documents describe a deployed configuration snapshot. They are not an
open-source release of the CourtRank implementation and do not grant a software
license.

## Provenance

This public archive was established on July 23, 2026 by extracting and
organizing publication artifacts and decision history from CourtRank's private
production project. Its short public commit history reflects that migration,
not the age of the underlying work. Historical paper versions are retained here
to make the algorithm's development traceable without publishing the
proprietary implementation or league data.

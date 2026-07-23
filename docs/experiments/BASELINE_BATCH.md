# Batch baseline (post-refactor) + re-validation

_Configuration fingerprint: generic engine defaults; weekly batch scoring;
`rank_clamp = 6`; `perf_blend = 0.6`; no 12-game pull taper. This validates the
batch semantic change, not the final production configuration._

Captured from `run_tests.py` **after** `run_week` was refactored to call the
batch `apply_week` (every game scores against the week-start snapshot). Compare
against `BASELINE_SEQUENTIAL.md`.

Env: Python 3.13, numpy 2.4.6.

## Scenarios (150 seasons each, locked defaults) — batch

```
A baseline (full attendance) wk1 rho=0.836  wk4=0.869  wk8=0.905  exact wk8=53.7%  +-1 wk8=92.1%  max up-move ever=6
B 12% weekly absences        wk1 rho=0.842  wk4=0.869  wk8=0.899  exact wk8=54.1%  +-1 wk8=92.5%  max up-move ever=6
C B + two wk-3 joiners       wk1 rho=0.840  wk4=0.845  wk8=0.885  exact wk8=51.0%  +-1 wk8=90.0%  max up-move ever=6
D sandbagger (top-3, last)   wk1 rho=0.739  wk4=0.827  wk8=0.879  exact wk8=50.4%  +-1 wk8=89.6%  max up-move ever=6
```

- Sandbagger avg court by week: `[6.00 5.57 5.06 4.46 3.79 3.31 2.95 2.70]`
- Misseeded joiner avg court (wk3+): `[3.86 3.55 3.14 2.67 2.28 2.23]`
- Rating drift / player / season: winner_floor ON = +0.1, OFF = -0.0
- **Guardrail: largest single-week upward rank move across ALL runs = 6 (cap=6)**

## Δ vs sequential — headline metrics

| Metric | Sequential | Batch | Δ |
|---|---|---|---|
| baseline wk1 ρ | 0.837 | 0.836 | −0.001 |
| baseline wk4 ρ | 0.873 | 0.869 | −0.004 |
| **baseline wk8 ρ** | **0.905** | **0.905** | **0.000** |
| baseline ±1-court wk8 | 92.2% | 92.1% | −0.1% |
| B absences wk8 ρ | 0.895 | 0.899 | +0.004 |
| C joiners wk8 ρ | 0.883 | 0.885 | +0.002 |
| D sandbagger wk8 ρ | 0.881 | 0.879 | −0.002 |
| **sandbagger wk8 court** | **2.78** | **2.70** | **−0.08** |
| drift | +0.1 | +0.1 | 0 |
| **guardrail max up-move** | **6** | **6** | **0 (exact)** |
| hidden-carrier gap (unit_checks) | +113 / 84% | +117 / 85% | negligible |

## Re-validation verdict — PASS

- baseline wk8 ρ ≈ 0.90 within ±0.02 → **0.905 (Δ 0.000)** ✓
- baseline ±1-court wk8 ≥ 88% → **92.1%** ✓
- sandbagger wk8 court ≤ 3.3 → **2.70** (climbs slightly faster than sequential) ✓
- guardrail max up-move == 6 exactly → **6** ✓
- drift within ±2 → **+0.1** ✓

The batch / order-independent model preserves every headline result and the
one-court guardrail. Differences are in the 3rd decimal (ρ) or < 0.1 court.

## `/courtrank` page reconciliation (Task 6) — verdict: no page change; flagged to owner

All rounded page claims hold under batch:
- "ρ 0.84 → 0.90" — batch baseline 0.836 → 0.905 ✓
- "92% within one court" — batch 92.1% ✓
- sandbagger "6 → ~3 by wk8" — batch 6 → 2.70 ✓

Chart-by-chart:
- **ConvergenceChart** — endpoints match within rounding (batch 0.836→0.905 vs page
  0.837→0.905; wk4 0.869 vs page 0.872). The page's 8-week array is an illustrative
  curve (the harness only emits wk1/wk4/wk8). **No change.**
- **LadderClimbChart** — the page's `SAND` array is the exact **sequential**
  trajectory `[6, 5.6, 5.1, 4.6, 4.0, 3.4, 3.0, 2.8]`; batch is
  `[6, 5.57, 5.06, 4.46, 3.79, 3.31, 2.95, 2.70]` (≈ `[6, 5.6, 5.1, 4.5, 3.8, 3.3, 3.0, 2.7]`).
  Four points differ by 0.1–0.2 court — batch climbs marginally faster. Barely
  visible (~5px on the chart).

**Decision:** make **no** page code change in M1. The page currently matches the
owner's **white paper** (also sequential); editing only the page would desync the
two, and the paper is owner-owned. If the owner wants the shipped (batch) numbers
reflected, refresh *both* the page chart `SAND` array and the white paper together
— a deliberate owner-driven follow-up, not a silent M1 edit.

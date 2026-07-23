# Sequential baseline (pre-batch)

_Configuration fingerprint: generic engine defaults; sequential within-session
updates; `rank_clamp = 6`; `perf_blend = 0.6`; no 12-game pull taper. Historical
baseline, not production._

Captured from `run_tests.py` against the vendored engine **before** the batch
refactor (`League.run_week` still applies games sequentially within a week).
This is the reference Task 3 compares the batch results against.

Env: Python 3.13, numpy 2.4.6. Reproduces the white-paper / `/courtrank` figures.

## Sanity grid (60 seasons/cell)

```
variant                            | wk1 rho wk8 rho  +-1 wk8 sand wk8
DEFAULTS (K=32 b=.5 bl=.6 t=16)    |   0.837   0.904    92.4%     2.65
K=24                               |   0.837   0.901    92.2%     2.82
K=48                               |   0.836   0.898    91.9%     2.52
beta=0.7                           |   0.828   0.889    91.1%     2.43
no perf pull                       |   0.848   0.905    91.8%     4.80
pull blend=0.8 tau=8               |   0.827   0.901    92.2%     2.97
```

## Scenarios (150 seasons each, locked defaults)

```
A baseline (full attendance) wk1 rho=0.837  wk4=0.873  wk8=0.905  exact wk8=53.5%  +-1 wk8=92.2%  max up-move ever=6
B 12% weekly absences        wk1 rho=0.843  wk4=0.864  wk8=0.895  exact wk8=53.7%  +-1 wk8=91.5%  max up-move ever=6
C B + two wk-3 joiners       wk1 rho=0.840  wk4=0.845  wk8=0.883  exact wk8=51.2%  +-1 wk8=89.7%  max up-move ever=6
D sandbagger (top-3, last)   wk1 rho=0.738  wk4=0.825  wk8=0.881  exact wk8=51.5%  +-1 wk8=89.7%  max up-move ever=6
```

- Sandbagger avg court by week: `[6.00 5.59 5.09 4.56 3.97 3.37 3.00 2.78]`
- Misseeded joiner avg court (wk3+): `[3.85 3.54 3.16 2.65 2.36 2.12]`
- Rating drift / player / season: winner_floor ON = +0.1, OFF = +0.1
- **Guardrail: largest single-week upward rank move across ALL runs = 6 (cap=6)**

## Headline references (Task 3 must hold within tolerance under batch)

| Metric | Sequential |
|---|---|
| baseline wk8 ρ | 0.905 |
| baseline ±1-court wk8 | 92.2% |
| sandbagger wk8 court | 2.78 |
| guardrail max up-move | 6 (exact) |
| drift | +0.1 |

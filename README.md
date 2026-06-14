# Fantasy Football Draft Optimizer

**MIT 15.C57 — Optimization Methods | Final Project**

This project applies integer programming and stochastic optimization to the problem of building an optimal fantasy football roster in a 12-team snake draft. Five drafting strategies are implemented and benchmarked against each other across multiple draft positions.

---

## Problem Setup

- **Format:** 12-team snake draft, 16 rounds (192 total picks)
- **Roster:** 2 QB, 5 RB, 5 WR, 2 TE, 1 K, 2 DST (starters: 1 QB, 2 RB, 2 WR, 1 TE, 1 FLEX, 1 K, 1 DST)
- **Scoring:** Standard ESPN fantasy scoring
- **Opponent model:** All 11 opponents draft in strict ADP (Average Draft Position) order
- **Data:** 533 players with 2025 projections, ADP rankings, upside/bust risk scores, and bye weeks

---

## Models

### Baselines
| Model | Description |
|-------|-------------|
| **ESPN Autodraft** (Baseline 0) | Selects players by ESPN platform rankings; represents a typical auto-draft user |
| **Myopic Greedy** (Baseline 1) | At each pick, takes the highest projected FPTS player available within roster limits |
| **Greedy w/ Reaching Penalty** (Baseline 2) | Solves a MIP at each pick over all remaining slots; penalizes drafting players significantly earlier than their ADP to avoid "reaching" |

### Optimization Models
| Model | Description |
|-------|-------------|
| **Static Global Optimization** (Model 1) | Single MIP solved over all 16 picks simultaneously; enforces ADP-consistency (players cannot be drafted after their ADP rank) and position limits |
| **Adaptive Optimization** (Model 2) | Extends Model 1 with bye week diversity constraints (no two RBs/WRs/TEs share a bye week) and upside/bust weighting in the objective |
| **Robust/Stochastic Optimization** (Model 3) | Simulates stochastic opponent behavior across multiple scenarios; solves a sample average approximation MIP to find a robust draft plan |

### Reaching Penalty (used in Baselines 2 and Models 1–2)
The penalty discourages taking players far before their ADP, scaled by round:

```
Penalty = λ × Excess_Reach × Pick_Value

Pick_Value = 400 × e^(-pick/80)      (early picks cost more)
λ = 1.0 (rounds 1–3), 0.5 (rounds 4–6), 0.2 (rounds 7–8), 0.02 (rounds 9+)
```

---

## Results (Draft Slot 7, Bye-Week Adjusted Points)

| Model | Starter Pts | Bye-Adjusted Pts | Rank |
|-------|-------------|------------------|------|
| ESPN Autodraft | 1822.8 | 1909.4 | 1 |
| Adaptive Optimization | 1774.2 | 1890.3 | 2 |
| Greedy w/ Penalty | 1782.0 | 1865.2 | 3 |
| Static Optimization | 1754.9 | 1857.8 | 4 |
| Myopic Greedy | 1657.0 | 1761.4 | 5 |

The bye-week evaluation runs a weekly lineup optimizer (another MIP) for all 18 NFL weeks, substituting bench players for starters who are on bye. The Adaptive Optimization model ranks highest among the custom models due to its built-in bye week diversity.

---

## Files

```
Opti_Proj/
├── Opti_Project_pick3.ipynb     # Full analysis — draft slot 3 (early pick)
├── Opti_Project_pick7.ipynb     # Full analysis — draft slot 7 (middle) + Robust Opt
├── Opti_Project_pick12.ipynb    # Full analysis — draft slot 12 (last pick, snake advantage)
├── Opti_Project_Draft.ipynb     # Earlier draft of analysis pipeline
├── output.csv                   # Player projections dataset (533 players, 80+ columns)
└── Opti_Project_Report.pdf      # Written report
```

The pick-3, pick-7, and pick-12 notebooks run the same pipeline at different draft positions to test whether optimal strategy changes based on where you pick in the first round.

---

## Dependencies

- **Julia** (≥ 1.9)
- **Gurobi** (academic license) — used via `JuMP.jl`
- **Packages:** `JuMP`, `Gurobi`, `CSV`, `DataFrames`, `StatsBase`, `Statistics`

To run a notebook, open it in Jupyter with the Julia kernel and ensure `output.csv` is in the same directory.

---

## Data

`output.csv` contains pre-season 2025 fantasy projections scraped and compiled from multiple expert consensus sources (ESPN, Sleeper, CBS, NFL.com, FantasyPros). Key columns:

- `PLAYER`, `POSITION`, `TEAM`, `BYE_WEEK`
- `FPTS` — projected fantasy points for the season
- `ADP_AVG` — average draft position across platforms
- `UPSIDE`, `BUST` — expert-assigned risk scores (1–5 scale)
- `ESPN`, `Sleeper`, `CBS`, `NFL` — platform-specific ADP rankings
- `2024_*` — prior season actuals for context

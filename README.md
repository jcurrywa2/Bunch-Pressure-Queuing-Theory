# Bunch Pressure Queue Theory (BPQT)

**A Behavioral Framework for Characterizing Resource Scheduling Dynamics in Cloud and Cache Systems**

> **The name derives from observing young soccer players surrounding the ball before learning positional strategy — aggressive players transmit pressure through the dense crowd, accelerating the entire group. The same phenomenon was independently observed in high-density boarding environments in China. Bunch Pressure Queue Theory formalizes this mechanism as an independent scheduling dimension.**

---

## Overview

Bunch Pressure Queue Theory (BPQT) introduces **behavioral aggression** as a new independent performance dimension orthogonal to priority in resource scheduling systems. Classical scheduling frameworks recognize one behavioral dimension: priority. BPQT formally defines aggression — the observed ratio of actual resource consumption to allocated resources — as a second independent classification dimension, providing the measurement and classification layer that current monitoring frameworks lack.

The framework identifies four behavioral quadrants based on the intersection of priority and aggression, with particular focus on the **Q4 rogue agent** (low priority, high aggression) that causes the noisy neighbor problem in cloud systems.

---

## Key Findings

Three simulation campaigns spanning **12,000+ independent runs** and Redis benchmark validation across **117 files and three independent runs of 200,000 operations** establish five key results:

1. **Optimal aggression improves passive latency 15-26%** across all tested scales (100 to 5,000 agents) at optimal aggression (85-90%), with simultaneous CPU idle reduction of 11-26 percentage points — confirmed across three independent 60-run trials.

2. **Continuous random weight assignment improves a further 19%** over discrete uniform assignment — behavioral diversity produces more efficient pressure propagation than discrete type categories.

3. **The self-sorting effect** — convergent random assignment where agents start with random weights and reclassify based on observed behavior — achieves **68% improvement** over orderly baseline using movement attempt rate as the convergence signal (V13). Agents with higher initial weights exit earlier, creating pressure waves that benefit remaining agents. The queue self-organizes without explicit scheduling intervention.

4. **Priority misalignment causes harm** — when low-priority agents behave aggressively, passive agent latency degrades 58-118%. The harm comes from misalignment, not from aggression itself.

5. **Redis benchmark validation confirms BPQT mechanisms in production** — throughput improves 16x at optimal pipeline depth (P=64), the variance signal predicts thrashing onset before throughput collapses (CV=67.9% at P=8), and the simulation-predicted optimal zone of 15-20% aggressive agents is confirmed in the mixed ratio sweep.

---

## Three Throughput Mechanisms

| Mechanism | Description | Evidence |
|---|---|---|
| Attentiveness Effect | Aggressive agents keep passive agents ready — eliminates phantom idle periods | CPU idle drops 11-26pp at optimal aggression |
| Pressure Propagation | Aggressive agents transmit forward pressure to adjacent passive agents | 15-26% passive latency improvement V11 |
| Self-Sorting Effect | Heterogeneous weight assignment causes natural queue stratification | 68% improvement V13 — Redis: 16x throughput at P=64 |

---

## Repository Contents

```
BunchPressureQueueTheory/
├── README.md                           — This file
├── LICENSE                             — CC0 1.0 Universal
├── paper/
│   └── BPQT_WhitePaper_v5_Final.docx  — Full research paper (two-column format)
├── notebooks/
│   ├── V1_CoreThroughput.ipynb         — Foundational orderly vs bunch comparison
│   ├── V2_PreRunAssessment.ipynb       — Capacity viability framework
│   ├── V3_ScaleTesting.ipynb           — Scale dependency 100-5000 agents
│   ├── V4_LatencyAnalysis.ipynb        — P50/P95/P99 latency by agent type
│   ├── V5_Animation.ipynb              — Visual validation of model dynamics
│   ├── V6_AggressionSweep.ipynb        — Scale-invariance, 2-80% sweep
│   ├── V7_TwoDimensional.ipynb         — Four quadrants, QoS enforcement
│   ├── V8_StepBudget.ipynb             — Optimal threshold stability
│   ├── V9_CrossoverExperiment.ipynb    — Passive latency at all scales
│   ├── V10_CPUIdle.ipynb               — CPU idle tracking introduction
│   ├── V11_Comprehensive.ipynb         — 10 aggression levels, 5 scales, 60 runs
│   ├── V12_RandomAggression.ipynb      — Random vs uniform vs convergent assignment
│   └── V13_AttemptRate.ipynb           — Attempt rate vs success rate convergence signal
├── figures/
│   ├── Fig1_FourQuadrant_BW.png        — Two-dimensional classification framework
│   ├── Fig2_PassiveLatency.png         — Passive latency vs aggression V11
│   ├── Fig3_CPUIdle.png                — CPU idle vs aggression V11
│   ├── Fig4_StdDev.png                 — Standard deviation thrashing signal
│   ├── Fig5_RedisBenchmark.png         — Redis concurrency sweep (combo chart)
│   └── Fig6_SimVsRedis.png             — Simulation vs Redis normalized comparison
├── data/
│   ├── v11_clean_results.json          — V11 three-run validated results
│   ├── v12_results.json                — V12 assignment method results
│   └── v13_results.json                — V13 attempt rate results
└── redis/
    ├── BPQT_Redis_Validated.xlsx       — All Redis benchmark data and charts
    └── redis_sweep_*.txt               — Raw benchmark files (117 files)
```

---

## Simulation Environment

| Component | Version |
|---|---|
| Python | 3.12 |
| Mesa (agent-based modeling) | 3.5.1 |
| NumPy | Latest stable |
| Matplotlib | Latest stable |
| OS | Ubuntu 24.04 LTS |

### Running the Notebooks

```bash
# Create virtual environment
python3 -m venv bunchqueue
source bunchqueue/bin/activate

# Install dependencies
pip install mesa==3.5.1 numpy matplotlib jupyter

# Launch Jupyter
jupyter notebook --notebook-dir=notebooks/
```

Open notebooks in order V1 through V13. V11 and V13 are the primary validation notebooks — allow 2-3 hours for the full scale sweep at 5000 agents.

---

## The Two-Dimensional Model

| Quadrant | Priority | Aggression | Role | Action |
|---|---|---|---|---|
| Q1 — Emergency | High | High | Legitimate fast path | Facilitate |
| Q2 — Starved VIP | High | Low | Displaced by rogue agents | Protect |
| Q3 — Background | Low | Low | Routine work | Serve when able |
| Q4 — Rogue | Low | High | Noisy neighbor | Detect and throttle |

### Behavior Score Formula

```
B = (P × w_p + A × w_a) × U / (S + E) × D × C
```

---

## Complete Performance Summary — 5000 Agents

| Approach | Passive Latency | vs Orderly | Notes |
|---|---|---|---|
| Orderly baseline (5%) | 1,273 steps | — | Reference |
| V11 Optimal (85-90%) | 940 steps | -26% | Three-run validated |
| V12 B-Random (85%) | 774 steps | -39% | Behavioral diversity |
| V12 C-Convergent (85%) | 507 steps | -60% | Self-sorting V12 |
| **V13 C-ConvAttempt (85%)** | **402 steps** | **-68%** | **Best result** |

---

## Redis Benchmark Validation

Three-run validated (200,000 operations × 3 runs):

| Scenario | BPQT Equivalent | Result | Confirmed |
|---|---|---|---|
| Passive baseline (c=1) | Orderly 5% | 1.0x | Reference |
| Optimal concurrency (c=50) | Bunch 15-20% | 10.6x throughput | ✓ |
| Pipeline optimal (P=64) | C-Convergent | 16.7x throughput | ✓ |
| Thrashing signal (P=8) | 90%+ boundary | CV=67.9% | ✓ |

---

## Citation

```
Curry, J. (2026). Bunch Pressure Queue Theory: A Behavioral Framework for
Characterizing Resource Scheduling Dynamics in Cloud and Cache Systems.
Working Paper v5.0. Provisional Patent Filed July 2026.
Zenodo. https://doi.org/10.5281/zenodo.21514017
```

*Replace XXXXXXX with your actual Zenodo DOI*

---

## Status

- [x] Provisional patent filed — July 2026
- [x] Simulation notebooks V1-V13 complete
- [x] Three-run validation confirmed — V11 and V13
- [x] Redis benchmark validation — 117 files, 3 runs each
- [x] White paper v5.0 complete
- [x] Zenodo preprint published — v5.0
- [ ] V14 — test at 50,000+ agents
- [ ] Production cloud validation
- [ ] Multi-node Redis Cluster benchmark

---

## License

CC0 1.0 Universal — see [LICENSE](LICENSE) for details.

## Contact

Jeffrey Curry — Retired Intel Engineer | Fire Commissioner | Volunteer Firefighter | Pacific Northwest, USA

*Research inquiries welcome.*

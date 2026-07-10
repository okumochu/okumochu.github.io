---
layout: page
title: "List Scheduling and Beam Search Methods for the FJSP with Sequencing Flexibility"
description: "Problem, method, and results of a strong non-learning FJSP baseline."
category: research paper
order: 10
---

---
## Motivation

- The classical **Flexible Job Shop Scheduling Problem (FJSP)** assumes operations within a job follow a **linear precedence** (chain). In many real-world settings (e.g., the printing industry), operations have **arbitrary DAG precedences** , some operations can run in parallel, and assembly/disassembly operations join or split sub-processes.
- This generalization is called **sequencing flexibility**: precedences between operations are given by an arbitrary **directed acyclic graph (DAG)** instead of a linear order.
- The extended FJSP with sequencing flexibility is **NP-hard** (it subsumes the classical JSP). Exact solvers (CPLEX) struggle on medium-to-large instances , many cannot be solved optimally even with 10 hours of CPU time.
- Very few heuristic methods exist for this extended problem. The authors aim to fill this gap with efficient constructive heuristics.

## Contribution

1. A **list scheduling algorithm** for the FJSP with sequencing flexibility , a non-hierarchical constructive heuristic that simultaneously selects operations, assigns machines, and determines start times in a single pass.
2. A **beam search method** that naturally extends the list scheduling algorithm by expanding multiple partial solutions at each step and using the list scheduler as a global evaluator for pruning.
3. A complete **benchmark suite** (YFJS01–YFJS20 and DAFJS01–DAFJS30) with 50 instances, plus full C/C++ source code for reproducibility.

## Problem Formulation

- **Input**: $$n$$ jobs, $$o$$ operations, $$m$$ machines. Each operation $$i$$ has a set of eligible machines $$F_i \subseteq \{1,\dots,m\}$$ with processing times $$p_{ik}$$. Precedences $$A$$ form a DAG over operations.
- **Decision variables**: 
  - $$x_{ik} \in \{0,1\}$$: operation $$i$$ assigned to machine $$k$$
  - $$y_{ij} \in \{0,1\}$$: sequencing of operations $$i,j$$ on the same machine
  - $$s_i \geq 0$$: start time of operation $$i$$
- **Objective**: Minimize makespan $$C_{\max} = \max_i (s_i + p_i)$$
- The MILP model (1–8) from Birgin et al. (2014) is used as the formal definition.

## Method

### List Scheduling Algorithm (Algorithm 1)

A greedy constructive heuristic that iterates $$o$$ times. At each iteration, it selects one operation, assigns it to a machine, and sets its start time using three sequential rules:

**Auxiliary data** (precomputed):
- $$P_i, S_i$$: predecessor/successor sets from DAG $$A$$
- $$\bar{p}_i$$: average processing time across eligible machines
- $$RW_i$$: remaining (blocked) work , longest weighted path from $$i$$ in the DAG

**Three selection rules** (applied sequentially to filter candidates):

| Rule | Criterion | Purpose |
|------|-----------|---------|
| **Rule 1** | Earliest starting time $$st_{ik} = \max\{u[i], v[k]\}$$ | Only keep pairs $$(i,k)$$ with minimum $$st$$ |
| **Rule 2** | Fastest machine per operation (breaking ties by machine load $$L[k]$$) | Remove redundant machine choices |
| **Rule 3** | Largest remaining work $$RW_i$$ (breaking ties by machine load, then index) | Prioritize operations blocking the most downstream work |

**Complexity**: $$O(\vert A\vert  + m + oq)$$ where $$q$$ depends on the max antichain width of the DAG.

### Beam Search Method (Algorithms 2–5)

A **filtered beam search** that extends the list scheduler by exploring multiple partial solutions in parallel at each tree level:

- **Local evaluation** (cheap): Apply Rules 1–3 iteratively with "forbidden pairs" to generate up to $$\hat{\beta} = \min\{\hat{\beta}_1, \hat{\beta}_2\}$$ children per node, where:
  - $$\hat{\beta}_1 = \lceil \beta \cdot \vert \Omega_0\vert  \rceil$$ , fraction of candidates  
  - $$\hat{\beta}_2 = \lceil \vert \Omega_{0.5}\vert  \rceil$$ , bounded by starting time tolerance $$\delta$$
- **Global evaluation** (expensive): Complete each child using the list scheduling algorithm; keep the child with smallest estimated $$C_{\max}^{est}$$.
- **Beam width**: At each level, keep $$\lceil \alpha \cdot \vert N\vert  \rceil$$ nodes (controlled by parameter $$\alpha \in (0,1]$$).
- **Duplicate elimination**: Identical partial solutions across different parents are detected and removed to maintain diversity.

**Three parameters**:
| Parameter | Role |
|-----------|------|
| $$\delta > 0$$ | Tolerance on earliest start time for candidate filtering |
| $$\alpha \in (0,1]$$ | Fraction of nodes kept at each level (beam width) |
| $$\beta \in (0,1]$$ | Fraction of children generated per node |

**Complexity**: $$O(r + oq^2\vert A\vert  + o^2q^3)$$ when $$\hat{\alpha}, \hat{\beta} = O(q)$$.

## Experiment

### Setup
- Language: C/C++, compiled with g++ (GCC 4.7.2, -O3)
- Hardware: Intel Xeon E5645 2.40GHz, 132GB RAM, Debian Linux
- Exact solver: IBM ILOG CPLEX 12.1 (1-hour and 10-hour time limits)

### Instances
- **YFJS01–YFJS20**: Y-shaped jobs (two independent sequences → assembling operation). 4–17 jobs, 7–26 machines, 24–289 operations.
- **DAFJS01–DAFJS30**: Arbitrary DAG precedences. 4–12 jobs, 5–10 machines, 25–127 operations.

### Key Results

**CPLEX exact solver**:
- Solved 14/20 YFJS instances optimally (1h limit); large instances had gaps up to 52%.
- Solved only 4/30 DAFJS instances optimally; gaps up to 67%.

**List scheduling algorithm**:
- Runs in < 1 second for all instances.
- YFJS: ~35% average gap vs. optimal (expected , most have known optima).
- DAFJS: ~5.7% average gap vs. CPLEX (1h) , actually *improves* on many CPLEX results because CPLEX didn't find optima.

**Beam search method** (80 parameter combinations tested):

| Instance Set | Best params $$(\alpha, \beta, \delta)$$ | Avg gap vs CPLEX (1h) | CPU time |
|---|---|---|---|
| YFJS01–YFJS20 | $$(1, 1, 1)$$ | **3.5%** | ~2,650 s/instance |
| DAFJS01–DAFJS30 | $$(1, 1, 0.5)$$ | **−6.36%** (improves CPLEX!) | ~115 s/instance |

- On DAFJS instances, beam search **outperforms** CPLEX (1h) by up to 17.5% on individual instances.
- Performance is robust: across all 80 parameter combos, gaps range only 3.5%–9.0% (YFJS).
- $$\delta \geq 0.5$$ is generally beneficial; larger $$\alpha, \beta$$ → better quality but more CPU time.

**Classical FJSP benchmarks** (no sequencing flexibility):
- Competitive with state-of-the-art GA methods (Pezzella et al., 2008), outperforming them on Dauzère-Pérès & Paulli and Hurink VData instance sets.

## Limitations & Thoughts

- The beam search is **deterministic** (no randomness), which is both a strength (reproducibility) and limitation (no diversification via restarts).
- Only **makespan** is considered; multi-objective extensions are not explored.
- Parameter tuning: while robustness is demonstrated, the best parameter triple varies by instance type ($$(\alpha, \beta, \delta) = (1,1,1)$$ for YFJS vs. $$(1,1,0.5)$$ for DAFJS).
- The beam search can be expensive for the best-quality settings (~2,650 s/instance for YFJS), though still much faster than CPLEX.
- The list scheduler's Rules 1–3 are intuitive heuristic choices but not formally justified as optimal local decisions.
- **Relevance to DRL-based scheduling**: The list scheduling structure (iterative operation/machine selection with state = partial solution) maps naturally to an RL formulation , the three rules could be replaced by a learned policy. The beam search could serve as a search strategy on top of a learned value function.

## Related Notes

- Combinatorial Optimization
- CP vs MIP

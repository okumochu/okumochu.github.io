---
layout: page
title: "Deep Reinforcement Learning for Dynamic Scheduling of a Flexible Job Shop"
description: "MDP breakdown plus my comparison tables against related dynamic-scheduling designs."
category: research paper
order: 4
---

https://github.com/RK0731/Deep-reinforcement-learning-for-dynamic-scheduling-of-a-flexible-job-shop?tab=readme-ov-file

本篇研究的方法是使用模擬的方式來訓練以及 evaluation。本篇研究的架構是不存在把 job 排完，而是設定一個模擬的時間，也代表只要一直有工作 arrive 他就能根據每個 machine 以及 排上去以及沒排上去的 job 的統計數據:

1. new job arrival 時，選擇一個 dispatching rule 選一個 job 到一個 machine 的 queue 上 (此時還沒排上去)
2. operation finished 時，選擇在該 machine 上下一個應該進行的 operation

最終希望能 Minimizing Tardiness

## Motivation
- 前面就講到說為什麼會有 dynamic events，並且我們為什麼要應對 dynamic envet。應對的方式有三種 proactive / predictive reactive / reactive
- 通常會用 priority rule (reactive) ，並且通常為了因應多目標會用 composite priority rule。然後各種 Meta-heuristic 方法都是用 proactive 的方法因為用 reactive 實在太花時間了
- 過去在處理的 dynamic events 通常都是 machine breakdown)。那如果非要 reschedule 的話，就會在 dynamic events 發生的時候用最快的速度把被影響到的排程做快速修復 i.e. priority rule
- 剛剛提到的方法就是他們會有一個收斂的過程使其造成 efficiency vs quality 的 trade-off。為了要快的並且更好的利用 shop floow information 排出好的排程，我們會使用 Genetic Programming、supervised learning、Reinforcement Learning
   - GP 使用來開發 dispathching rule 的 (找出一個 dispatching rule 會最小化 symbolic regression error、clasification accuracy、control policy performance 之類的)
   - ANN (supervised-learning) 通常是用在選擇 priority rule (沒有細看是怎麼做的，但是他說 The NN parameters are determined through simulation optimization)

> **Note**
> This research adopts the most general context – flexible job shop for better transferability and comparability

## Methodology

### State
本研究有要解決 fixed size 的問題的，在 state representation 上選則了統計的方式來統一表示，而不是選擇一個 job 一個 job 來表示

> **Note**
> 這邊要注意，job 的數量可以不用 fixed size 沒有關係，但是 machine 的數量要 fixed，直接看下面的 information of machine 就知道了。如果 machine 數量變了就一定要重新 train RA

<img src="{{ '/assets/notes/drl-dynamic-fjs/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/drl-dynamic-fjs/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

### Action
Routing Agent 會從下面的四個 dispatching rule 選一個出來用

- **Shortest Processing Time (SPT)**: select the job with shortest processing time of imminent operation.
- **Work in Queue (WINQ)**: select the job with the smallest sum of processing time of queuing jobs in its succeeding stage of production. WINQ rule tends to even the distribution of jobs in the system.
   - 簡單來說就是看一在還沒排的工作裡面，假設排上去要等多久才做。挑那個最不用等的 (平衡 machine utilization)
- **Critical Ratio (CR)**: select the job with smallest ratio of TTD to remaining processing time. A disadvantage of CR rule is its inconsistency. When available jobs are not yet tardy, CR rule favors jobs with smaller TTD and longer remaining processing time. On the other hand, if available jobs are already tardy, CR rule prioritizes the job with highest overdue time (smallest TTD) and shortest remaining processing time.
   - $$\frac{\text{Time Til Due} }{\text{Remaining Process Time} }$$
- **Minimum Slack (MS)**: select the job with smallest slack time. This rule behaves like CR rule if jobs are not yet overdue, favoring jobs with shorter TTD and longer remaining processing time (shorter slack time as a result) and remains consistent when the job becomes tardy.

### Reward
本篇研究提到 reward 可能會有兩個問題
1. multi-agent 都用相同的 reward 來訓練是沒有問題的，但是這樣當reward 變大就無法判斷是哪個 agent 的貢獻。這也被稱為 multi-agent credit assignment problem
2. 有可能會有 bias on different scenario 像是如果是 SPT rule 可以在很多很趕的工作的 arrive 的時候快速減少 tardy job 和 tardiness，但是犧牲的就是 maximum tardiness。也就是說這個 signal 要**能真正 optimize primary objective value**，不然 agent 就會學歪一直去吃 shaped reward

所以本研究特別設計 shaped reward 給兩個不同的 agent，然後主要是用能**保有或是消耗多少 slack time** 作為 reward shaping 的設計主軸，同時這個值會乘上一個權重，這個權重是估計 preserve slack time 的難度

**Routing Agent**
- difficulty weight $$\times$$ (real slack time $$-$$ expected slack time)
   - expected slack time 的計算方式就是先看選訂的 job 還有多少時間可以加工 $$-$$ 剩餘的工作時間 $$-$$ 平均機器等待時間 (要排隊)
   - 直觀來看就是 slack time 比期望來說好了多少

**Sequencing Agent**
- 選擇的 operation 消耗的 slack time $$-$$ 因為這個 operation 被選消耗的 slack time
   - 直觀來說就是這次選擇要做的 operation 是否對整個系統的 slack 有幫助 (正的就是有幫助)

### Asynchronous transition
這個兩個 RL agent 都是 event trigger 並且是不同的 event，所以他們並不會一起同時 take action

<img src="{{ '/assets/notes/drl-dynamic-fjs/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

## Experiment
The experiments are entirely **simulation-based** (SimPy). There is no real dataset; both training and evaluation run on the authors' own dynamic FJSP simulator. The campaign is structured into three escalating stages: (1) Independent utility test, (2) Validation of integrated DRL, (3) Scalability test.

### Configuration
- Each simulation episode lasts **100,000 time units**, during which roughly **12,400 jobs arrive** (Poisson stream, non-terminating simulation).
- Base shop has **2 machines** with a fixed number of operations per job.
- Four scenario combinations from a $$2^2$$ design are evaluated independently:
   - Process-time variability: **high vs. low**
   - Due-date tightness: **tight vs. loose**
- To mitigate **multi-agent non-stationarity**, the two agents are trained in an **alternating** fashion: when one agent is being updated, the other is **frozen**.
- All input features go through **layer (instance) normalization** to stabilize training.
- DRL / ANN hyperparameters and network architectures:
   <img src="{{ '/assets/notes/drl-dynamic-fjs/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
   <img src="{{ '/assets/notes/drl-dynamic-fjs/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

- Overall simulation-and-training procedure (alternating training of RA and SA on top of a shared production simulator):
   <img src="{{ '/assets/notes/drl-dynamic-fjs/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />

### Independent utility test
> **Note**
> RA and SA are **validated separately first** to confirm that each component delivers a benefit on its own, before combining them.

Two metrics are reported:
1. **% improvement**:  improvement in tardiness over a chosen baseline rule (used as denominator).
2. **Win rate**: fraction of replications in which the proposed agent beats the baseline.

- **RA + FIFO vs. each routing rule + FIFO** (sequencing held constant at FIFO so routing differences are isolated)
   - Benchmark routing rules:    <img src="{{ '/assets/notes/drl-dynamic-fjs/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
   - Finding: as process-time variability grows, RA's advantage becomes larger, because routing decisions have more leverage when machines differ more.

- **Earliest Available (EA) + SA vs. EA + each sequencing rule** (routing held constant at EA so sequencing differences are isolated)
   - Benchmark sequencing rules:
	    <img src="{{ '/assets/notes/drl-dynamic-fjs/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
   - Finding: SA wins on average, fixed dispatching rules do not generalize well across the four scenario combinations.

### Validation of integrated DRL
> **Note**
> RA and SA are deployed **together**. To assess synergy, the half-versions **RA + FIFO** and **EA + SA** are also benchmarked alongside the full **RA + SA** combination.

**Baselines:**
1. Top-2 routing rules $$\times$$ top-5 sequencing rules (= 10 hand-crafted combinations).
2. Two composite rules produced by **Genetic Programming**.
3. Two prior **DRL** methods:

   > Lang, S., Behrendt, F., Lanzerath, N., Reggelin, T., & Müller, M. (2020). Integration of deep reinforcement learning and discrete-event simulation for real-time scheduling of a flexible job shop production. *2020 Winter Simulation Conference (WSC)*, 3057–3068.

   > Luo, S. (2020). Dynamic scheduling for flexible job shop with new job insertions by deep reinforcement learning. *Applied Soft Computing, 91*, 106208.

   | Element | Lang et al. (2020) | Luo (2020) |
   | --- | --- | --- |
   | **Problem setting** | Static FJSP (all jobs known up front) | Dynamic FJSP (random job insertions) |
   | **Agents** | 2 (sequencing + machine assignment) | 1 (composite rule selector) |
   | **State design** | "Machine buffer workload" + LSTM operation-sequence representation | 7 scaled generic features + statistical aggregation |
   | **Action space** | Directly outputs Machine ID or Sequence ID | Selects from 6 composite dispatching rules |
   | **Reward** | Local buffer/earliness reward + terminal $$-(C_{\max}+T)$$ | Slack-difference local reward + terminal $$-\text{Total Tardiness}$$ |

> **Note**
> Meta-heuristics (GA etc.) are **excluded** from comparison because they are too slow to reschedule in real time under dynamic events.

**Conclusions of this stage:**
- RA + SA **outperforms** every baseline category.
- The half-versions (RA + FIFO, EA + SA) lose to the full version, confirming a true **synergy**: both agents are needed.
- **Routing contributes more than sequencing** to the overall gain (likely because once routing is decided, the sequencing search space is already constrained).

### Scalability
Two orthogonal stress tests are performed.
- **Higher-flexibility test**: vary the number of machines.
   - Increase machines from 2 to **3 and 4**.
   - Because RA's input dimension depends on the number of machines, **RA is retrained** with a larger input; **SA is reused as-is** (its statistical features are size-insensitive).
   - Finding: as the shop becomes more flexible, tardiness improvement grows, but **win rate drops**. The authors attribute the drop to the relative **change in job arrival rate** (more machines without proportionally more arrivals), not to a weakness of the method.

- **Longer-sequence test**: vary the number of operations per job.
   - Increase operation count to **6 and 9**.
   - **SA is retrained** (its input depends on sequence length); **RA is reused as-is**.
   - Finding: the proposed method's advantage does **not** grow much with longer sequences, indicating that the sequencing approach is **size-insensitive**, consistent with the classic observation in:
      - Holthaus, O., & Rajendran, C. (1997). Efficient dispatching rules for scheduling in a job shop. *International Journal of Production Economics, 48*(1), 87–105.

<img src="{{ '/assets/notes/drl-dynamic-fjs/Pasted-image-20260425154644.png' | relative_url }}" alt="Pasted image 20260425154644" loading="lazy" style="max-width: 100%; height: auto;" />
## Related Notes
- Apparent Tardiness Cost (ATC) scheduling rule

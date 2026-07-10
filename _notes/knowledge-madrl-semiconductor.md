---
layout: page
title: "Knowledge-enabled Multi-agent DRL for Batch Machine Scheduling in Semiconductor Fabrication"
description: "IRL plus MADRL pipeline, reward design, and my critique of it."
category: research paper
order: 8
---

## Introduction

- **Batch scheduling + dynamic event (wafer arrival) + re-entrant constraint** → Find scheduling time in acceptable time is important
   - **Heuristic (rule-base)** and **Metaheuristic** is primary approach to batch scheduling
   - Remains a limitation in terms of their ability to **update in real time**
- DRL could solve the problem perfectly in real-time, but in scheduling problem, it suffered form **credit assignment problem** (actions (or decisions) deserve how much of the final overall reward)
   - Design a reward function to signal agent **before the true objective revealed** is important
   - use **Inverse reinforcement learning** to ****extracting knowledge from data to guide the design of reward functions
      - Extract from vast collection of near-optimal solutions of this problem

### Contribution

1. Extract the demonstration trajectory $$\tau = \{s_t, a_t, s_{t+1},...\}$$ from **Hybrid Ant Colony Optimization (HACO)** serving as a  ****near-optimal solution and the raw data for Inverse-RL (IRL)
2. Propose a **Bidirectional LSTM–Guided Cost Learning (BLSTM-GCL)** IRL network to find the **relationship** of **current state parameter** (batch size, current flow time, expected remaining flow time, success-flag) and **objective function** (we would know the immediate reward using this four parameters)
   1. **Maximum-entropy guided cost learning** to extract reward function (estimate the cost using these four shop floor parameters)
        
      $$ r_\psi(s,a)\;=\;\sum_{i=1}^4 \omega_i\,p_i(s,a) $$
        
3. After getting data-driven reward function
   1. Construct two actor critic agent
      1. **Batch-forming agent**: decides when and which pending lots to group into a batch
      2. **Scheduling agent**: assigns ready batches to idle machines
   2. Training two agent, they introduce a **central GRU collaboration module**
      1. At each decision step, the agents feed their chosen action and observed state into the GRU, which encodes a shared “informing vector” summarizing **all prior scheduling context**.
      2. Both actors and critics take this vector plus their local observation as input, allowing **dynamic, history-aware coordination**.
      3. Training uses a modified Trust-Region Policy Optimization (TRPO) that backpropagates through the GRU to align each agent’s policy with the global objective

## Problem Description

**Objective: Minimize average flow time of a wafer batch** 

$$ \overline{F} \;=\; \frac{1}{n} \sum_{i=1}^{n} F_i\;=\;\frac{1}{n} \sum_{i=1}^{n} \bigl(C_i - r_i\bigr). $$

- Problem Description
   - N wafer lots arrive over time and are formed into multiple batches, where the number of lots in each batch shouldn’t exceed the maximum capacity B
   - The process type for wafer lots within the same batch should be identical, which requires batch machines to use the same process parameters (temperature, time, chemical environment, etc.) for all wafers when processing a batch
   - Wafer lots are subjected to multiple process steps, and these are often multi-layered, meaning that lots may need to be processed multiple times on the same or different machines (so-called re-entry flow)
   - It is important to note that a batch machine can only process batches of one type at a time. The batch machines are composed of two or more equivalent parallel machines, and these machines operate in a non-preemptive manner, meaning that once they commence processing, they cannot be interrupted.
   - In addition, some production lead time is required when switching process types on batch machines, e.g. switching from one process to another may require cleaning and reconfiguration of the equipment.
- Problem Assumption
   1. Batches arrive dynamically, and process type and processing time information is available for each layer of each lot during processing
   2. Machine failures and delays due to maintenance are not considered
   3. The processing time of a batch is independent of the number of lots comprising the batch, and is related only to the process type of the batch.
   4. Batch buffer capacity in front of batch machines is unlimited
   5. Batch machines have a maximum capacity limit
   6. Different production lead times are incurred when processing different process products before and after the batch machines
   7. The batch cannot be interrupted once processing has begun, and is a non-preemptive process
   8. There are no emergency order insertions, and batches are strictly enforced in accordance with the scheduling solution

<img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> Each wafer lot must pass through a fixed sequence of processing layers in the fab
> Think of each layer as one operation (e.g. oxidation, deposition, etch) that the lot must undergo. The index $$w$$ simply denotes which layer in that sequence

> **Note**
> 每個 lot 接下來的加工 (layer) 要相同才能 batch 在ㄧ起
> 然後若該 batch 仍有後續加工，就會整批 lots arrive → batch forming → scheduling

## Methodology

<img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

**State**

- **Batch-forming agent** observes the state of **lots** and **batches**
- **Scheduling agent** observes the state of **batches** and **machine**
    
   <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

**Action**

- **Batch-forming agent**
   - Select Lot number for batch forming (list of lot-indices)
   - Do nothing
- **Scheduling agent** observes the **batches** and **machine**
    
   When ever a machine is idle
    
   - Assign a batch to that idle machine
   - Do nothing

**Reward**
DRL agents will use a **learned weight** to formulate reward (current flow time / subsequent expected flow time / batch size / batch-forming success flag)

 $$r(s,a)\;=\;\omega_1 p_1 + \omega_2 p_2 + \omega_3 p_3 + \omega_4 p_4$$

- **Batch-forming agent**
    
   $$r_2 = -\Bigl[\omega_1\,(t - a_{ti}) + \omega_2\,F_{i,w} + \omega_3\,V_j\Bigr]\;s_{\omega_2}(t)\;-\;\omega_4\bigl(1 - s_{\omega_2}(t)\bigr)$$
    
   - $$t - a_{ti}$$ : current flow time of the lots waiting in buffer
      - $$a_{ti}$$ is the time when the lots fist arrive
   - $$F_{i,w}$$ : subsequent expected flow time
   - $$V_j$$ : batch size
   - $$s_{\omega_2}(t)$$: 1 if a successful batch formation just occurred, 0 otherwise
- **Scheduling agent**
    
   $$r_3 = -\bigl(t - S_{mk}\bigr)$$
    
   - where $$S_{mk}$$ is the time the machine became idle. so it would penalize forced idle time

> **Note**
> Global reward function 就是一個很基本的 critic 架構。我覺得想表達的是透過這個 Basic RL 的設計 (discount factor) 做到避免 greedy。 但是我覺得有點沒新意

### **Learn** a reward via BLSTM-GCL IRL

> To establish the **relationship** between the **key feature parameters** in scheduling and the **global optimization objective**


<img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

### Guide Cost Learning

It is an algorithm of **Inverse Reinforcement Learning (IRL)**

Finn, C., Levine, S., & Abbeel, P. (2016, June). Guided cost learning: Deep inverse optimal control via policy optimization. In *International conference on machine learning* (pp. 49-58). PMLR.

- 簡單來說他的做法就是
   1. 選定一個專家 policy (state action pair) 
   2. 根據目前的 estimated reward function 學習一個 policy
   3. 他們會有各自的 trajectory，然後再用相同的 estimated reward function 去計算並相減，這個值必須越大越好
   4. Back propagation for reward function 然後回到第二步，直到收斂
    
   > **Note**
   > 最大化這個目標函數會使前面這項變大，後面這項變小。很合理，因為一個好的 estimated reward function 的結果應該會顯示出專家的 policy 會比這個 estimate reward function 訓練的 policy 來的好，所以差距要變大
   > 在 iteration 中期 estimated reward function 還會越來越準，使得其 estimate reward function 訓練的 policy 越來越像 專家的 policy 使其差距縮小
   > 這個過程就一職重複直到收斂 (reward 拉開差距 → policy 往上追趕 → 再拉開差距 → 再追趕)
    
   - 下面這個就是要 maximize 的 objective function ($$\tau$$ = trajectory)
        
      $$ L(\psi) \,=\, \frac{1}{N}\sum_{i=1}^{N} r_{\psi}(\tau_i) \;-\; \frac{1}{M}\sum_{j=1}^{M} r_{\psi}(\tilde\tau_j) $$
        

### **Train** two DRL agents

> The GRU network is utilized to enable collaborative interaction between the batch forming agent and the scheduling agent, aiming for global optimization by minimizing the average flow time of wafer batches


<img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

- 中間會有一個 global GRU network，作為兩個 agent 一部分的 observation，另一部分就是他們自己的 local observation。叫做 informing vector，加入這個能使 RL agent 更有記憶功能
    
   $$ m_{dc-1} \;=\;\mathrm{GRU}\bigl(h_{dc-2},\,[\,S_{dc-1},\,a_{dc-1}\,];\,\psi\bigr) $$
    
- Update Algorithm
    
   <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Numerical Experiment

note: all state would go through min-max normalization

Experiment Instance

- **Time horizon:** Simulated 1 year, with a 2-month warm-up (no data collected), then data collected over 2 months, updated monthly
- **Lot scales:**
   - *Small scale:* 2 months of lots, arrivals ∼ Normal(mid-month, 15 days).
   - *Large scale:* 6 months of lots, arrivals ∼ two Normals (10th & 20th day, 10 days variance).
- **Machine/process scales:**
   - Two process types (diffusion, wet etch)
   - Two machine configurations (9 machines vs. 14 machines)

- Scheduling knowledge extraction
   - Outputs every 50 steps a new set of **feature-weight vectors $$\psi=[\omega_1,\dots,\omega_4]$$**
   - 當小規模時， current flow time 跟 subsequent expected flow time 權重成反相關
    
   <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Multi-agent training
   - Configuration
        
      <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
        
    
   <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Performance Comparison (random parameter)
   - Experiment Configuration
        
      <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - RL 跟 HACO 表現差不多
   - KE‐MADRL 在多數實驗中獲勝，且因為有 Guiding 所以減少試錯，訓練時間明顯下降
    
   <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-10.png' | relative_url }}" alt="image 10" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Performance Comparison (simulation instance)
    
   > To further validate the effectiveness of the proposed algorithm, **a fabrication simulation model was established** based on the batch processing area of a semiconductor enterprise, **using the simulation software Plant Simulation 9.0**. In the simulation model, the distribution of wafer dynamic arrivals is varied to maximize the restoration of real production scenarios
   The model is set to run for 1 year, with the first 2 months being the transition period from initialization to stable production of the system, with no data collection; the first data collection starts after 2 months, with a 2 month cycle for the first data collection, and the data are updated once a month thereafter

   - A1在某些例子上是佔優勢的，但是 generalization 並不強
   - KE-MADRL 在大部分例子上都是贏的
        
      <img src="{{ '/assets/notes/knowledge-madrl-semiconductor/image-11.png' | relative_url }}" alt="image 11" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Conclusion

- Reward Function 和 reward shaping 的 signal 的設計非常困難，設計不好可能會導致 sub optimal policy。所以本研究使用 BLSTM-GCL IRL 抽取 HACO 的 near-optimal solution 來設計 reward function
- MADRL model with batch forming agents and scheduling agents is designed , incorporating a GRU-based collaboration mechanism to enhance single scheduling accuracy

## Related Notes

- Proximal Policy Optimization (PPO)

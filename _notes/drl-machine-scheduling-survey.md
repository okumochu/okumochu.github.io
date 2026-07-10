---
layout: page
title: "Deep RL for Machine Scheduling: Methodology, State of the Art, and Future Directions"
description: "Taxonomy of state, action, and reward designs across the machine-scheduling literature."
category: research paper
order: 14
---

- Machine Scheduling is a challenging **combinatorial** optimization problem (NP-Hard)

## Introduction

- Good Scheduling Algorithm
   - Meet all the operational constraints
   - Solution Quality
   - Computational efficiency

### Scheduling Algorithm

- **Exact Solver**  (Branch and Bound, cutting-plane)
    
   > Search solution space based on **enumeration**

   - **Strength**: good performance under **small-scale problem**
   - **Weakness**: computation time increase dramatically with the complexity of problem increase
   - Issue: Handle the **balance** between **solution quality** and **computation time**
- **Heuristic** (Priority Dispatching Rule)
    
   > Assign a value to each waiting job based on specific rule, and make decision based on that value

   - **Strength:** Interpretable, Fast
   - **Weakness: Lousy solution quality, due to lacking of adaptability to specific scheduling scenarios**
- **Meta-Heuristic** (Genetic Algorithm)
    
   > problem-Independent problem. neighbor search to find solutions

   - Slower then Heuristic, but often get **good solution quality** under even **large-scale problem**
- **Deep Reinforcement Leaning**
   - **Strength**: being scalable and  generalizable in dynamic and complex environments

## DRL approaches for machine scheduling

Separate DRL approach with four different **computational component (state → action)**

1. **Conventional DRL:** MLP, RNN, CNN
2. **Advance DRL**: Encoder-Decoder, GNN
3. **Hybrid DRL-Meta-heuristic:** DRL + meta-heuristic

- **State**
   - Statistic-base, Matrix-base, Graph-base, RNN-base, Transformer
- **Action**
   - rule-base action: select a dispatching rule from several rules (SPT, EDD, …)
   - attribute-based action: describe jobs attribute, and choose the closest (process time, setup time, due date)
    
   > **Note**
   > 以上兩種方法都不用管 job 的數量是多少，good for scalability。但是這兩個方法都要有 domain knowledge 才能設計出真正有用的
    
   - job-based action: direct probability to the job

### Hybrid

> many of these **algorithms require exhaustive fine-tuning**, more specifically for the number of populations or other mutation-related parameters. In such a case, some recent papers have proposed frameworks in which an **RL agent** tries to **tune these parameters** in a systematic manner


## Application

這裡有提到很多在不同問題上 (single, parallel, flow shop, job shop, …) 用不同的架構 (single agent, multi-agent, Hierarchical RL) 不同的演算法 (NASH Q learning) 處理不同的 Event (machine breakdown) 以及 constraint (setup time)

<img src="{{ '/assets/notes/drl-machine-scheduling-survey/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

- DRL 在簡單的環境下不太能取得優勢，因為使用 meta-heuristic 或是 數學模型即可求出最佳解
- 考慮到環境上會有些限制 soft constraint (penalty), hard constraint (mask)
   - e.g. sequence dependent setup time, limited buffer size, machine breakdown
- 76 % 單一目標 34 % 是多目標問題。再多目標問題上都用 weighted sum generate single optimal solution, no Pareto optimal set
   - 這樣訓練出來的模型只能用在特定目標值權重上，但也有人在訓練中讓目標值權重變換使得其 agent 能在不同的目標值權重上使用
- 設計 reward 時，有些 objective (通常會拿來設計成 reward) 在排程結束才會拿到 objective (e.g. makespan)。所以通常都是利用 reward shaping (machine utilization) 使其在排程中途也有 reward
- 在 multi-agent 上會有幾種做法
   - One machine per agent, One job per agent, One for machine selection + One for job sequence, etc
   - Centralized Learning: involves a **global network** that **aggregates local
   agent experiences**, allowing agents to access broader environment in-
   formation and enabling parallel training
   - Decentralized Learning: trained independently with local information
- **Hierarchical DRL 也有在用但是還不多 (5)**
   - **High level policy:** abstract goal / **Low level policy:** choose dispatching rule, machine, job
- Agent Capability
   - **Generalization**: be able to produce optimal schedules for **unseen instances**
   - **Scalability**: be able to handle **large-scale scheduling problems**
    
   > **Note**
   > 以上這兩個能力，能使模型穩健且不需要 retrain or redesign
    
- Environment
   - **Static setting:** the agent schedules all jobs based on initial information at the start of the scheduling horizon
   - **Dynamic setting:** it manages scheduling in a predictive-reactive manner, few for fully reactive manner
    
   > **Note**
   > few studies have incorporated **robustness** against unexpected events and disruptions into their DRL models (e.g. job delay, machine breakdown)
    

## Future Direction

- Scalable deep reinforcement learning
   - job shop, flexible job shop, and open shop: only solve small and medium problem size
    
   > Adopted a bi-level hierarchical DRL model where the high and low level were controlled by a DDQN network and GPN, respectively. Their framework could optimize flexible flow shop instances with 5000 jobs and 50 machines.

- Incorporation of manufacturing constraints
   - e.g. machine eligibility, limited auxiliary, resource availability, job release time
   - Solutions
      - Hard: incorporate constraint during schedule construction (e.g. adjust DRL framework, Masking)。簡單來說就是直接在排程機制上就不讓他有機會產生違反 constraint 的狀況 (infeasible solution)
      - Soft: penalize it in reward function
- Configurable multi-objective optimization
   - 過去的都是固定 objective weight，使其退化成有點退化成單一目標問題
   - 如果所有 non-dominated solution 都要找到 (formed the Pareto frontier)，一個一個訓練需要很多時間
- Robustness and resiliency
   - 現實中的排程會遇到很多 dynamic events，如何有很好的機制應對是非常重要的
   - 現在的研究通常是 purely **reactive** or **predictive-reactive**: focus on maximizing efficiency during rescheduling
   - **proactive** scheduling: predicting uncertain events in advance and mitigating their impact proactively，可能會有 force idle time 犧牲一點效能。(這個就像我們的 robust tft-lcd scheduling)
      - These studies trained DRL agents under **stochastic condition**
        
      > Addressing robustness in a proactive manner remains an open research challenge

- Integrated maintenance planning and machine scheduling
- Real-life implementation (要真正在現場使用)
- Learning from demonstration
   - 將人類專家的排程結果蒸餾出來
   - e.g. Imitation Learning, Inverse Reinforcement Learning (learn reward function from demonstration)
- Online learning / Transfer Learning / Meta-DRL
   - 需要在環境變動的時候，你的模型仍然能夠使用的方法

## Related Notes

- Combinatorial Optimization
- Proximal Policy Optimization (PPO)

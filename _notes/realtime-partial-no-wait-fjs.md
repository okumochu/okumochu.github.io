---
layout: page
title: "Real-Time Scheduling for Dynamic Partial-No-Wait Multiobjective Flexible Job Shop by DRL"
description: "Hierarchical multi-agent MDP with rule tables and Pareto metrics."
category: research paper
order: 12
---

## Problem Definition

- Job environment:  Flexible Job Shop
   - New job insertion
   - Machine break down
   - Partial no wait

- Multi-objective
   - Total weighted tardiness
   - Average Machine utilization
   - Variation of machine workload (workload balance)
        
      <img src="{{ '/assets/notes/realtime-partial-no-wait-fjs/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        

- Hierarchical Multi-Agent RL
   - Objective **Manager**
        
      <img src="{{ '/assets/notes/realtime-partial-no-wait-fjs/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - ( Machine + Job ) **Worker**

## Reinforcement Learning

> **Note**
> 本研究使用的排程機制就是透過事件三種事件的發生來做一次 worker 的決策，並且每 C 次 manager 的給的 $$O_{temp}$$ 會調整一次 
>
> 這三種事件就是 new job arrival / machine break down / machine idle
> note: 如果某 operation 在做的時候剛好那台 machine breakdown 不會重新做，而是指定修好後直接把剩下的 process time 做完

### State

- 總共 10 個數值
    
   For reschedule point = $$t$$
    
   - number of machine
   - machine utilization rate
   - completion rate of **operation** & **job**
   - std of job completion rate
   - **average** and **std** of ****normalized machine workload (divided by max workload)
   - of normalized machine workload
   - estimated tardiness rate
      - 估計這個 operation 是否 tardy 會先看這個 job 的目前的排程上 (在 reschedule point $$t$$) 的 operation 做到哪裡
      - 若已經做完就直接取完成時間，若還正在做就取該 machine 的估計完成時間
      - 把前面的完成時間 $$+$$ 到做完該 operation 的預估處理時間
         - 預估處理時間就是每個的 operation 的 process time 除上 available machine
      - 最後判斷是否超出 operation 是否會 Due date
   - actual tardiness rate
      - 已經 tardy 的 operations 除上還所有**尚未排程的 operations**
    
   > **Note**
   > 以上的兩個 tardiness 都是算 rate 所以都會除上 **尚未排程的 operations 個數**
   > 想要計算的是在接下來所有尚未完成的工作中，有多少比例的 operation **預估**會 tardy，或是目前看來實際上已有多少比例 tardy
    
- 然後此時刻的 10 個數值 $$V_t$$ 跟 $$V_t - V_{t-1}$$ 接起來當作這一期的 state $$S_t$$  加總為 20 個數值
   - 理由是因為 “state 的改變” 的這個資訊，可以反映出 performance 是否有 improve，這對於 agent 來說很重要
        
      > Citation: Design, implementation and evaluation of reinforcement learning for an adaptive order dispatching in job shop manufacturing systems

        
      $$ S_t = [V_t, \; V_t - V_{t-1}] $$
        

### Action

根據目前的 $$S_t$$ 和 每 $$C$$ 步都會更新一次的 temporal objective $$O_{temp}$$ 來選出兩個 rule

> **Note**
> 由於這邊的 rule 是手動定義的，我其實不太喜歡這種做法。我認為這種 rule ，主要目的是可以增加收斂性，但是彈性是比較低的。

**Job Selection Rule (5 rules)**

- Detail of Job Selection Rule
    
    
   | 規則 | 選擇邏輯 | 主要考量／想達成的效果 |
   | --- | --- | --- |
   | 1. **Due-date 逼近度／剩餘工序數** | 若此時沒有預估遲交的工作，挑選「到期日剩餘時間 ÷ 尚未加工工序數 × 1/權重」最小者；若已有遲交風險，挑選「加權預估遲延量」最大的工作。 | 兼顧 **緊迫性**（臨近到期日）與 **工作長度**；對高權重（高罰則）工作給較大優先權。 |
   | 2. **Due-date 逼近度／剩餘加工時間** | 與規則 1 類似，但分母改用「剩餘總加工時間」。 | 精細化評估剩餘負荷，特別適合各工序時間差異大時。 |
   | 3. **動態 Slack / 加權遲延** | 比較「目前平均完工時間 + 剩餘加工時間 − 到期日」。若尚未遲延，以原值排序；若已遲延，乘上權重後再排序，取最大者。 | 同時考慮提早完工空隙 (slack) 及遲延嚴重度，高權重工作更易被優先處理。 |
   | 4. **進度落後度 × Slack** | 若無遲延風險：以「已完成比率 × Slack × 1/權重」最小者為先；若有遲延風險：以「(1/已完成比率) × Slack × 權重」最大者為先。 | 把 **已完成進度** 納入考量,,越晚開始、進度越落後的急件會被提早拉到前面。 |
   | 5. **隨機抽選** | 從待加工工作集中隨機挑一件。 | 提供探索性 (exploration) 行為，讓 DRL 在訓練過程仍有機會嘗試其他可能。 |

**Machine Selection Rule (6 rules)**

- Detail of Machine Selection Rule
    
    
   | 規則 | 指派邏輯 | 主要考量／想達成的效果 |
   | --- | --- | --- |
   | 1. **Earliest Available Machine** | 把工序派到「最早空閒時間」的機台；若有 *no-wait*，續派相關工序。 | 儘快開工，減少等待時間，可降低整體遲延。 |
   | 2. **Lowest Utilization** | 派到目前平均利用率最低的機台。 | 均衡負載、提升整體機台利用率 ($$U_{ave}$$)。 |
   | 3. **Lowest Workload** | 派到累積已排產工時最少的機台。 | 減少工作量標準差 ($$W_{std}$$)，讓各機台工作量更均衡。 |
   | 4. **Shortest Processing Time Machine** | 選擇對該工序（或連續工序包）加工時間最短的機台。 | 縮短該批工序加工時間，間接壓低遲延風險。 |
   | 5. **Earliest Finish Time** | 考慮機台可用時間＋工序時間，派到「預計完工最早」的機台；若異機台 *no-wait*，同步計算下一工序。 | 直接最小化單批工序完工時間，對減少加權遲延最有效。 |
   | 6. **Random Machine** | 在可行機台中隨機選一台；若有 *no-wait*，相關工序同邏輯隨機。 | 提供探索性，避免長期陷入單一負載分配模式。 |

**No-wait constraint**

| 情況 | 具體作法 |
| --- | --- |
| **同機台 (homogeneous) no-wait:** 一段連續工序必須在同一機台、不間斷完成 | 直接將 $$O_{i,j}$$ 及其接下來需要連續執行的 operation「打包」，交由 **Machine agent** 一次性指派到**同一台**機器 |
| **異機台 (heterogeneous) no-wait:** 相鄰兩道工序須立即切到另一台機器 | 若第下一台機器在預計接續時刻還忙，則**把前一道工序整體往後移。**讓下個 machine 能夠接續進行 **(一次選兩個 machine 在時間不間斷的排上去)** |

### Reward

- Objective Agent 指定目前要聚焦的暫時性目標 $$O_{\text{temp} }$$
   - 其實這個 $$O_{temp}$$ 很直觀就是選一個下面的目標: action space = 3。選好之後這 $$C$$ 個 reschedule point 就會以這個目標當作 reward function
   - Expected TWT, Average machine utilization, Workload balance
- 在每個 reschedule point $$t$$（新工作插入、機台修復或機台變空閒時）完成排程後，即刻計算指標 (當前 objective agent 決定聚焦的指標) 在 $$t$$ 與 $$t+1$$ 的變化量 (= $$r_t$$)，並依下列規則給予獎勵
    
    
   | 目標 | 改善門檻 | $$r_t=+1$$ | $$r_t=0$$ | $$r_t = -1$$ |
   | --- | --- | --- | --- | --- |
   | $$ETWT$$ | 任意降低 | $$\text{ETWT}$$ 明顯下降 | 無變化 | 上升 |
   | $$U_{\text{ave} }$$ | 3 % | 利用率提高 | 提高≲3 % | 明顯下降 |
   | $$W_{\text{std} }$$ | 10 % | 負荷標準差降低>10 % | 下降≤10 % | 上升 |

> **Note**
> 1. 有設門檻的: 為了避免波動
> 2. 設定 reward = {-1, 0, 1} 是避免三個目標尺度不同的問題
>
> 這兩個 worker agent 並沒有溝通的機制，兩個 worker 都是看到相同的 state 並且吃相同的 reward。這應該就是因為這兩個 agent 並沒有競爭關係所以 reward 的方向一致

| 層級 | 更新節奏 | 觀測的同一個即時獎勵 $$r_t$$ 如何累積？ |
| --- | --- | --- |
| **Job & Machine agents** (下層) | 每 $$M$$ 步更新一次 ($$M$$ 是更新頻率的超參數) | 直接用最近 $$M$$ 步的 $$r_t$$ 組成 n-step 回報 |
| **Objective agent** (上層) | 每 $$C\!\times\!M$$ 步更新一次 | 先把連續 $$C$$ 個 reschedule point 的 $$r_t$$ 以折扣因子 $$\gamma$$ 加總成回報 $$G$$，再計算優勢值 |
- **Whole Scheduling Process**
    
   <img src="{{ '/assets/notes/realtime-partial-no-wait-fjs/screenshot_2025-06-13_143312.png' | relative_url }}" alt="screenshot_2025-06-13_143312" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Experiment

- HMAPPO 1000 episode
- 每個週期都在隨機生成的環境中進行訓練 (Robustness + Generalization)

**Performance Matrices**

由於真實的 Pareto solution 不知道，用可能想到的方法多次運行，並找出所有的 non-dominant point 建構 Pareto Frontier

- Generational Distance: 目前的解離 Frontier 有多近
- Inverse Generation Distance:
    
   其中 $$P$$ 為 frontier ，$$A$$ 為演算法產生的解集。IGD  = 每一個真帕累托點到最近近似解的距離再平均。**同時衡量收斂度（越接近真前緣）與多樣性（前緣覆蓋範圍），值愈小愈好**：IGD = 0 表示近似前緣完全貼合真前緣
    
   $$ {\small\text{IGD}(P, A)=\frac{1}{|P|}\sum_{p\in P}\min_{a\in A}\|p-a\|} $$
    
- Spread: 衡量這個方法得出的解在 Frontier 上的分佈 (多樣性指標)

> **Note**
> 為什麼不用直接加權算出總分數，要用 Pareto Frontier 的概念
> 當你在評估你的演算法是否 robust 的時候應該要再不同的權重組合下都能跑出好結果，而不是只有在某些特定組合而已。除非在實務上就已經確定權重就是這樣

### Sensitive Analysis of Parameter C

這個 $$C$$  代表調整 Manger 調整 objective 的頻率，每 $$C$$ 步就會調整一次頻率；較小的 $$C$$ 能更快的應對 uncertainty 但是會導致排程不太穩定。 $$C$$ = 20 是最佳

### Compare with Existing Method

**Dispatching Rule** 

- FIFO、EDD、MRT、SPT、LPT、CR
- 本研究提出的方法 outperform

**Real-Time Scheduling Methods** 

- HEFT、GRASP、Double DQN (這些方法都能在一秒內重新排程)
- 本研究提出的方法雖然在 Generational Distance 指標大家都做的不錯，但是在後兩個指標本研究的方法表現比較好

**Meta-heuristic Methods with Rolling-Horizon**

- 多目標粒子群優化演算法、NSGA-II
- 本研究在**時間上**以及解的品質上 outperform

## Related Notes

- Proximal Policy Optimization (PPO)

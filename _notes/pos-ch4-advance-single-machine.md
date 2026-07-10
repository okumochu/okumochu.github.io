---
layout: page
title: "Ch 4 Advance Single Machine"
category: lecture
course: "Production and Operations Scheduling"
order: 4
---

## Total Earliness and Tardiness

$$1\vert \vert \sum_j E_j + T_j$$

> **Note**
> 目前探討的 objective 其實都是 regular，更多的是更難的問題 e.g. $$\sum{E_j}+\sum{T_j}$$。所以先考慮特例

考慮一特例所有人 $$d_j =d$$

- 所有排程的job可以被分成
   - 能在 $$d$$ 完成的job $$J_1 = \{j\vert C_j \le d\}$$
   - 跟不行的job $$J_2 = \{j\vert C_j-p_j \ge d\}$$
- 對於上述的問題最佳解有以下特性
   - $$J_1$$  滿足 Longest Processing Time First
   - $$J_2$$ 滿足 Shortest Processing Time First
   - Proof
        
      <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
    

**With Loose Due Date** 第一個工作不一定要在 time = 0 開始

存在一optimal sequence，使得其中一個job是剛好在 time = $$d$$ 時完成

- Proof
    
   因為兩邊的在 optimal sequence 下 process time會差不多，所以只需要數個數就可以了
    
   <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Algorithm
   - 主要想讓兩邊的job process time平衡 **(保證最佳)**
      - 按照process time降序排序，大的擺前面 (影響力最大到最小)
      - 並交錯分配給 $$J_1$$ and $$J_2$$
   - 但若問題強制 t = 0 開始的話就是NP-Hard

**With Tight Due Date (NP-Hard)**

Heuristic Algorithm:

- process time先由大到小排好
- $$\text{set } \tau_1 = d, \tau_2 = \sum_jp_j - d, k =1$$
   - 測量該 process的開工時間 ~ $$d$$ 的距離 = $$\tau_1$$，和 process 完工時間 ~ $$d$$ 的距離 = $$\tau_2$$
      - $$\text{if } \tau_1 > \tau_2$$  then job $$k$$  放在最前面 (留空間給其他人)，then $$\tau_1 = \tau_1 - p_k$$
      - $$\text{else}$$ job $$k$$ 放在最後面，then $$\tau_2 = \tau_2 - p_k$$
   - $$k = k+ 1$$
   - next round

**Total Weighted Earliness and Tardiness**

use WLPT / and WSPT

## Multiple Objective

**Goal programming:** 會以主要的目標作為最佳化目標，當最佳化的結果不單一獨特的，就會運用次要的目標作為solution的中選擇

Lexicographic Optimization

> Lexicographic optimization is a method of multi-objective optimization where objectives are ranked in order of importance, and the optimization is performed sequentially - first optimizing the primary objective, then using any remaining degrees of freedom to optimize secondary objectives.


 $$\min \sum_j C_j, \min L_{\max} \rightarrow Lex(\sum_j C_j, L_{\max})$$

- e.g. Lmax = primary objective: 就會產生很多不同的解，但Lmax都相同。total completion time = primary objective: 運用SPT就可以解完，所以可能相同的目標值得解就沒有這麼多
- 如果先用EDD找到 $$L_{max}$$ ，就可以把所有人的 $$d_j$$ 加上這個數值，再把問題變成讓所有人都在 $$d_j + L_{max}$$ 的這個時限內做完 **(due date → deadline)**
- 對於一個 $$Lex(\sum_j C_j, L_{\max})$$ 在單一機台上的問題，總共有 n 個job，每個job都能在時限前做完，存在一個 job $$k$$ 排在最後能最小化 $$\sum C_j$$ 。若且唯若，這個 job $$k$$ 的process time最大且所有 job 能在 deadline 前做完。
   - Proof
        
      <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/Screenshot_2025-04-02_at_2.31.26_PM.png' | relative_url }}" alt="Screenshot_2025-04-02_at_2.31.26_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - Algorithm
        
      <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/Screenshot_2025-04-02_at_2.44.30_PM.png' | relative_url }}" alt="Screenshot_2025-04-02_at_2.44.30_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Pareto-Optimal Schedule

> A schedule is called Pareto-optimal if it is not possible to decrease the value of one objective without increasing the value of the other.


會畫出一個前緣，在前緣外的都不是好的解，前緣內的是目前沒辦法做到的。

<img src="{{ '/assets/notes/pos-ch4-advance-single-machine/Screenshot_2025-04-02_at_2.51.42_PM.png' | relative_url }}" alt="Screenshot_2025-04-02_at_2.51.42_PM" loading="lazy" style="max-width: 100%; height: auto;" />

### Find all the Pareto Optimal Schedule

For bi-objective problem = $$\sum C_j, L_{max}$$

- Algorithm
   1. First Generate the first Pareto Optimal Schedule with EDD
   2. 把你的到期日轉換成deadline 
      1. for the first iteration, it would be calculate using EDD
      2. 接下來的 iteration 都是加上 $$\delta$$
   3. 找一個 job $$j^*$$ ，能在時限前完成且處理時間最大，放在最後
   4. 找出其他還沒排進去的 job ，檢查是否 process time 比 $$j^*$$ 還大。如果比較大就要把差值紀錄為 $$\delta^\prime$$ ，接下來跟上回合的取最小值 $$\delta = min(\delta, \delta^\prime)$$ 
      1. 這個 $$\delta$$ 會作用在於紀錄在所有回合下如何最小的使 $$L_{max}$$ 變差但是使 $$\sum C_j$$ 變好
      2. Get the minimum increment of $$L_{max}$$ needed to minimize $$\sum C_j$$
   5. 一直排排到所有 job 排完之後，回到 step 2，準備找出下一個柏拉圖最佳解
    
   > **Note**
   > 基本上是在用 SPT with deadline constraint 來解每個 iteration，但是每個 iteration 結束之後就會放鬆一點 deadline 讓同一個 iteration 中不同回合有多一點的選擇可以選。到最後就是完全放鬆每個都可以選也就 pure SPT
    
- Example
    
   <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/Screenshot_2025-04-07_at_1.28.35_PM.png' | relative_url }}" alt="Screenshot_2025-04-07_at_1.28.35_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/Screenshot_2025-04-07_at_1.28.51_PM.png' | relative_url }}" alt="Screenshot_2025-04-07_at_1.28.51_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Sequence Dependent Setup Times

$$1 \vert  s_{j, k} \vert  C_{max}$$ 是一個 strongly NP-Hard 的問題

但是當 $$s_{j, k}$$ 有特定形式的時後

- $$s_{j, k} = \vert a_k-b_j\vert $$ 從 j ~ k 就像是從 state a ~ state b，並且到 state b 之後就會留在  state b。第一個 和最後一個 job 也都有setup time
- 實務上來說這個很常見 e.g. 溫度、機器參數等
- 在這個特殊形式下存在多項式時間的解 **Glimore and Gomori**
- 也可以使用 Heuristic Rule (Shortest Setup Time First, SST)
   - 等價於 TSP problem 的greedy algorithm which is nearest neighbor
   - Example
        
      <img src="{{ '/assets/notes/pos-ch4-advance-single-machine/Screenshot_2025-04-03_at_10.13.23_PM.png' | relative_url }}" alt="Screenshot_2025-04-03_at_10.13.23_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Batch Processing

A machine can process a number of job simultaneously

The process time = the longest process in the batch

- batch size = 1，就等同於沒有batch
- batch size = infinite ( ≥ n )，這種狀況其實很常發生。當機台數很多，但單很少時

When batch size = infinite, objective is regular then SPT-batch schedule is optimal

- SPT-batch schedule
   - Minimize $$L_{max}$$

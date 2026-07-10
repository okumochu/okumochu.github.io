---
layout: page
title: "Flexible Job-Shop Scheduling via Graph Neural Network and Deep Reinforcement Learning"
description: "Detailed method and experiment summary of a key heterogeneous-graph baseline."
category: research paper
order: 5
---

he proposed method also has good practical value. Its neural architecture is **size agnostic**; hence, the trained policy can be applied to solve the instances of varying sizes, not only the training size.

## Motivation

State representation 很重要，過去使用的方法大多都是把資訊疊乘一個 matrix 然後丟進 MLP (或是比較 fancy 的會丟進 CNN)。

- 這些方法的缺點就是 dimension 會是固定的，問題的 size 變了就不能用了必須重新訓練
   - 我認爲其實這也跟 action space 有關，action space 也必須要 size-agnostic
- RL 主要採取的動作應該要有兩個: 要做哪一個 operation / 指派到哪一個 machine 上。所以在 state 的表示上應該要更加的有資訊量 (彼此的 connection)，**尤其是 Flexible Machine**
   - 在表示 FJSP 的圖的時候會有以下元素
      - 一個 Start 和 End  代表開始和結束
      - Conjunctive arcs (單向) 代表的是 operation 的順序
      - Disjunctive arcs (會組成 clique) 代表的是 machine 要處理 operation 的順序。這些是虛線，當順序被決定後才會變成實線 (如右圖)
        
      <img src="{{ '/assets/notes/fjsp-gnn-drl/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
      > **Note**
      > 不像 JSP，FJSP 是可以有比較彈性的 disjunctive arc。通常 JSP 的圖就是同一個 operation index 就會有 disjunctive arcs 連在一起

        

## Reinforcement Learning

<img src="{{ '/assets/notes/fjsp-gnn-drl/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

### State

**heterogeneous graph**

- Homogeneous graph vs Heterogeneous Graph
    
   **Heterogeneous Graph 的 Node 和 Arc 會有不同的意義**
    
   | 特性 | 同質圖 | 異質圖 |
   | --- | --- | --- |
   | 節點種類 | 單一 | 多種 |
   | 邊種類 | 單一 | 多種 |
   | 鄰接矩陣 | 單一矩陣 | 多張關係矩陣，每種邊類型一張 |
   | 舉例 | 社交網路 (Node 代表人；Arc 代表朋友關係) | 學術網路 (Node 可能有作者或論文；Arc 可以是引用或是撰寫) |
- 保留 Operation node 和 Conjunctive arcs，然後移除 disjunctive arcs，取而代之的是 undirected arcs connect  **Machine node** (new added) and **Operation node** (有連起來到表該 operation 可以交由該 machine 處理)
    
   <img src="{{ '/assets/notes/fjsp-gnn-drl/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

本篇研究 proposed **heterogeneous graph 因為以下原因**

1. 如果直接用 disjunctive graph 來表示的話，當面對 large scale 的問題的時候就有可能會因為太多 disjunctive arcs 使整張圖變得非常 dense 難以處理
2. 多一個 machine node 可以放 machine information，同時 $$p_{ijk}$$ 也能直接表示在 O-M arcs 上

**Heterogeneous GNN: two stage embedding**

> **Note**
> 論文提到通常 HGNN 比較多在處理的是 node 的 feature，比較少處理 arc 的 feature (arc 都直接設距離為 1，然後看要走幾個邊來到達另一個 node 來判斷距離遠近)

Feature’s Symbol Definition

- **Operation =** $$\mu_{i,j} \in \R^6$$
   1. scheduled flag
   2. number of neighboring machine (compatible machines)
   3. processing time 
   4. Estimated or actual start time
        
      Estimate 的方式就是透過上一個 operation 的結束時間加上這個 operation 預估的 process time (process time / 所有可用的 machine) = 下一個 operation 的開始時間
        
   5. number of unscheduled operation in this job
   6. job-level completion time estimation
- **Machine =** $$v_k \in \R^3$$
   1. machine available time
   2. number of neighboring operaions (opreation that is compatible for machine)
   3. utilization (nonidle time / total production time) range = [0, 1]
- **O-M** **arc =** $$\lambda_{i,j} \in \R$$
   1. each arc $$E_{i,j,k}$$ =corresponding process time
- **Operation embedding = $$\mu_{i,j}^\prime \in \R^d$$ / Machine embedding** = $$v_k^\prime \in \R^d$$
- **Graph Representation** = $$H_t=(O,\;M,\;C,\;E_t) \in \R^d$$
   - $$E_t$$ undirected operation-machine (O–M) arcs that exist **only** for machines still **compatible with** (and **not yet chosen for**) each operation

1. **First Stage: Machine Node Embedding**
   - 採用 Graph Attention Network (GAT) ， 因為可以學到 node 彼此之間的重要程度。根據本研究提出的圖可以知道 machine 連接到的只有 operation 所以 GAT 可以學到 operation 對於 machine 來說有多重要
   - 但是 GAT 是設計給 homogeneous graph 來使用的，所以本研究做了修改
      - 在 Attention 中，會使用相同的 W 來做 projection e.g. $$[Wx_i‖Wx_j]$$
         - 因為是 homogeneous 所以 node 的 feature 都一樣
      - 所以改用 $$W^{M} \in \R^{d\times3}$$ 和 $$W^{O} \in \R^{d\times7}$$
         - Operation Feature dim = 7 是因為把 process time 的資訊也放進去

1. **Second Stage: Operation Node Embedding**
   - 總共有以下幾種資訊
      - 前面和後面和自己本身各一個總共三個的 operation node 的 information
      - 還有 $$N_t(O_{i,j})$$ 中多個鄰居 machine 的資訊，直接做加總
   - 把上面的資訊全部 concatenate 通過 ELU (activation function)
   - 最後再過一個 MLP  投射成 $$d$$ 維，輸出 $$\mu^\prime_{i,j}$$

1. **接下來把每一層的 embedding 做 mean pooling**
   - 剛剛的兩個步驟就像是在下圖的 HGNN，把 operation 和 machine 的 raw feature 轉換到 embedding space。這兩個 raw feature 只有第一層會用到
   - 但是 raw $$\lambda_{i,j,k}$$ 則是每一層都會用到，就直接把 dimension + 1 然後放進去。e.g. 6d → 7d
   - 最後得到一個 $$h_t \in \R^{2d}$$
    
   > **Note**
   > 這邊的 pooling 也是 size-agnostic 的關鍵，因為他把所有的 machine 和 operation mean pooling 起來 (一個黃條或藍條就是一個 operation 或 machine node)。這樣就沒有不同個數的問題了
   >
   > note: 根據公式要取出 GNN 的 embedding 要看是要取第幾層起來做 pooling，每一層 value = 鄰居的 information 加上自己的 information。這也代表只要 feature 數量沒有變就不會 dim 上的問題，因為進去 GNN 後是什麼 n*n 出來就是什麼 n*n*L (L 是不同的層)
    

1. **最後一步就是 policy 的 input = $$[\mu_{i,j}^\prime \vert \vert  v_k^\prime \vert \vert  h_t]$$**
    
   然後過一個 MLP → softmax 取出最大的那個
    

<img src="{{ '/assets/notes/fjsp-gnn-drl/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

$$\text{MLP}\phi$$ = critic network: predict $$v(s_t)$$
這邊也有把 Pooling 前該 Operation 和 Machine 的 embedding 抽出來給 policy 作為 input (這樣也比較合理，因為 Pooling 完就全部混在一起了)

<img src="{{ '/assets/notes/fjsp-gnn-drl/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> 看到下圖有 layer 跟 GAT 的 layer 相同的意思，這篇研究設定 L = 2 其實是有他的道理的
>
> Machine Embedding
> Layer = 1 時會得到的是 machine 到 $$\mu_{11}$$ 和 $$\mu_{22}$$ 以及 $$\mu_{12}$$ 的關係
> Layer = 2 時會得到的是 $$\mu_{11}$$  和 $$\mu_{22}$$ 以及 $$\mu_{12}$$ 的關係
>
> Operation Embedding
> Layer = 1 時會得到的是本身 operation 到 前後 operation 和 machine 的關係
> Layer = 2 時會得到的是前後 operation 和 machine 彼此的關係
>
> Note: 不管現在是第幾層，計算 attention 時，仍然是拿鄰居的邊算。只是這個過程會隨著我計算的次數變多，很遠的點的資訊會被傳遞過來

### Action

在 step = $$t$$ 時，把每一個可行的 $$(O_{ij}, M_k)$$ 整理下來然後一一過 policy network 做評分，最後就會得到所有可行動作的評分，最後過一個 softmax 變成一個機率分佈

- 可行是指在 step = $$t$$ 時，前面 operation 已經做完的 $$O_{i,j}$$ 以及目前 idle 的 machine $$M_k$$

> **Note**
> 這個 policy 是用來打分的，所以不受 action space 限制。所以總共的可行的 $$(O_{ij}, M_k)$$ 可以變大變小，但是仍然選的出來。
>
> 缺點: inference 時間線性成長、如果 graph 的資訊有 shift，原本訓練好的 feature extraction 能力就有可能失效

### Reward

- 每一步的 reward 就是 Estimated $$C_{max}(s_t) -$$ Estimated $$C_{max}(s_{t+1})$$
- 並且設定 discount factor = 1，所以 cumulative reward = $$C_{max}(s_0) - C_{max}$$
   - $$C_{max}(s_0)$$  是一個 constant

### Scheduling Procedure

<img src="{{ '/assets/notes/fjsp-gnn-drl/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />

- Training Detail
    
   > **Note**
   > Actor 跟 Critic 基本上是相同的架構，但是 input space 不同。Actor: 4d / Critic: 2d
    
   <img src="{{ '/assets/notes/fjsp-gnn-drl/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
    
- 每當有一個 operation 完成時，就會 trigger policy 排一個上去然後進入 $$s_{t+1}$$
- 每次有 action 執行後就會代表該 Operation $$O_{i,j}$$ 已經做完，所以就會把 $$O_{i,j}$$ 其他的邊刪掉
- 直到所有 operation 都做完

> **Note**
> 這篇也是沒有 forced idle time

## Experiment

### Training & Testing Set

- Configuration Table
    
   <img src="{{ '/assets/notes/fjsp-gnn-drl/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
    
   job 的 average process time 抽出後，再從 $$U(0.8 \times \bar{p}_{i,j}, 1.2 \times \bar{p}_{i,j})$$ 抽出每個 operation 的 process time
    

**synthetic instances**

well-known benchmark

> Brandimarte, P. (1993). Routing and scheduling in a flexible job shop by tabu search. Annals of Operations research, 41(3), 157-183.


**well-known FJSP benchmarks**

以下的這些問題的 parameter 的 distribution 都非常的不一樣，所以很適合用來測試 generalization 的能力

1. ten mk instances (mk01–mk10)
    
   > Brandimarte, P. (1993). Routing and scheduling in a flexible job shop by tabu search. Annals of Operations research, 41(3), 157-183.

2. three groups of la instances (rdata, edata, and vdata, each with 40 instances)
    
   > Hurink, J., Jurisch, B., & Thole, M. (1994). Tabu search for the job-shop scheduling problem with multi-purpose machines. Operations-Research-Spektrum, 15, 205-215.


> **Note**
> 本篇研究適用小的問題訓練，然後在大問題上 testing (30 × 10 and 40 × 10)
> 然後拿每次訓練就 sample 100 個不同的 training 環境，測試也是 100 個 testing 環境拿來測平均效能
>
> Note: 中間有一個部分我覺得滿特別的，他在訓練不同的環境下時，我推測每次到新的環境的時候都會 cumulative reward 會很震盪。但是如果每次都用 100 個 validation set 來測試 cumulative reward 可以讓他不會這麼震盪的同時也可以真正的測出 generalization 的能力 (而不是單純看 training 的 reward 有沒有下降的比較快)

### Baseline Method

Minimize Makespan

- PDR: FIFO、MOR、SPT、MWKR
- Exact Method: Google OR tool (30 min)
- Meta-Heuristc:
   - Self-Learning Genetic Algorithm: 於搜尋過程中用強化學習自動調整 GA 參數，以提升解品質
   - Two-Stage Genetic Algorithm
- Proposed DRL:
   - DRL Greedy
   - DRL Sampling

### Experiment Result

1. 在相同的大小的問題上，能贏其他的 PDR 但是輸解 30 min 的 OR-tool
   - Performance Table
        
      <img src="{{ '/assets/notes/fjsp-gnn-drl/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />
        
      <img src="{{ '/assets/notes/fjsp-gnn-drl/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />
        
2. 在小問題的下訓練出來的 policy 可以在更大的問題上使用。直接將 20 × 10 policy 套用至 30 × 10、40 × 10，仍全面領先 PDR。但是仍然輸解 30 min 的 OR-tool
   - Generalization Table
        
      <img src="{{ '/assets/notes/fjsp-gnn-drl/image-10.png' | relative_url }}" alt="image 10" loading="lazy" style="max-width: 100%; height: auto;" />
        
    
   > **Note**
   > 不過為什麼不反過來做 (訓練在難的問題上)，反過來做不是更有用處嗎?
    
3. Run time 上略輸 PDR，但是贏 OR-tool
   - Run time Table
        
      <img src="{{ '/assets/notes/fjsp-gnn-drl/image-11.png' | relative_url }}" alt="image 11" loading="lazy" style="max-width: 100%; height: auto;" />
        
4. 在 other well-known benchmark 就是可以看到就是把前面在不同排程複雜度訓練好的 policy 拿來用 (10x5, 20x5, …)。實驗上來說比 GA、OR-tool 差 (本篇論文 argue that search base 要花比較多時間)，然後比 PDR 好。
   - Well-known benchmark Table
        
      <img src="{{ '/assets/notes/fjsp-gnn-drl/image-12.png' | relative_url }}" alt="image 12" loading="lazy" style="max-width: 100%; height: auto;" />
        
    
   > **Note**
   > 這個實驗在測試的是 parameter 的 distribution shift 後能不能依舊能保持效果。也就是所謂的這個 policy 是有真正學會排程，而不是硬背 training set。結果看上來 generalization 的能力是可以的
   >
   > 從這邊就可以發現一個比較違反直覺的就是，既然他有 generalization 的能力，代表真的在學習排程。但是在訓練在比較複雜的環境下不會明顯的使效果變好 (20x10 沒有明顯比 10x5 厲害)。
   >
   > 這篇論文的最後有說明，為什麼 10x5 最厲害
   >
   > > The training of the DRL agent could be more sufficient so that it can discover
   > better patterns. We will investigate this in the future.
   > > 
    

> **Note**
> 最後 future work 的時候有 cite 一篇論文 (Policy Optimization with Multiple Optima)
> 把層次拉到能解多目標 **Combinatorial Optimization** 的問題在同一個最小化或最大化目標下 **(單目標) ，**盡可能探索並找到所有同樣優秀 (objective value 相同) 的解。
>
> Multiple Optima ≠ Multi-Objective
>
> [POMO: Policy Optimization with Multiple Optima for Reinforcement Learning](https://proceedings.neurips.cc/paper/2020/hash/f231f2107df69eab0a3862d50018a9b2-Abstract.html)

## Related Notes

- Proximal Policy Optimization (PPO)
- Combinatorial Optimization

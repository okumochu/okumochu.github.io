---
layout: page
title: "Ch 3 Single Machine Method"
category: lecture
course: "Production and Operations Scheduling"
order: 3
---

> **Note**
> 在 single machine problem 下
>
> schedule without idle time的排程就會形成 dominant set
>
> - Proof
>
>     <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-06_at_2.25.49_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_2.25.49_PM" loading="lazy" style="max-width: 100%; height: auto;" />
>
>     - 接下來把  $$S_b$$ 全部往前移，就會發現 $$S_a$$ 沒有變，但是所有排在 $$S_b$$ 的 job 的 $$C_j$$ 變小了。使得 $$Z^\prime \le Z$$
>     - 所以只要移除 idle time 就可以使得

## Completion Time

### Total Completion

用 sorting algorithm 就可以做完了 $$O(nlogn)$$

Shortest Process Time (SPT) is optimal to $$1\vert \vert \sum{C_j}$$ 

- Proof
    
   藉由反證法 (假設有一個排程是最佳解但是沒有按照這個規則)
    
   - 有一個排程 $$S$$ 有最小的 total processing time，同時有一個 job $$i$$ 排在 job $$j$$ 前面且 $$p_i < p_j$$
   - 令一 $$S^\prime$$ 交換了 job $$i$$ 和 job $$j$$ 其他排序都一樣
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-06_at_3.26.34_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_3.26.34_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - 所以 $$Z_{S^\prime} < Z_{S}$$
   - 故所有 processing time 比較小的都要排前面

**Total weighted Completion Time**

Weighted Shortest Processing Time first (WSPT) is optimal ****to $$1\vert \vert \sum W_jC_{j}$$

- Proof
   - 用反證法，如果相鄰個 process（前面後面process 固定）， $$w_j / p_j$$ 比較大的先做，會導致目標值變差
   - 證明法跟 SPT 幾乎一樣

### Maximum Cost

> $$h_j(C_j)$$ 是一個 non-decreasing with time 的一個 function，也就是說隨著時間增加這個 function只會變大不會變小。Maximum Cost 即為 $$h_{max} = \max(h_j(C_j)) ,\space \forall j$$


General Form = $$1 \vert prec\vert h_{max}$$ 

- Backward Dynamic Programming
   - Lowest Cost Last (LCL) is optimal
   - Algorithm
        
      精神: 因為 cost function $$h$$ 是隨時間上升，所以如果最後排的一定是所有 job 的 cost function 的最大值也非常有可能就是 $$h_{max}$$。
        
      1. 找出所有沒有 successor 的 job
      2. 從最後一個位置開始，計算每個 job 的 cost function 的值
      3. 剛剛算出來最小的排最後面
      4. 找下一輪沒有 successor 的 job 一直這樣排，排到全部排完
   - Proof
        
      根據反證法
        
      1. 找到一個最佳排程 $$S$$，並且其中有兩個 job，在 position $$k$$ 時 cost function 比另一個 job 小但他沒有排在那個位置
      2. 發現  $$h(x) ≥$$ 把 cost function 比較小的排在 position $$k$$
      3. 又目標為最小化  $$h_{max} = \max(h_j(C_j)) ,\space \forall j$$
      4. 於是總是要把當前位置 cost function 最小的排上去

### Precedence Constraint

**Select A Chain from multiple Chains**

- 那哪條 chain 做 → 把兩條的 $$\sum w_j / \sum p_j$$ ，大的先做 (注意，是上面加上面，下面加下面)
    
   <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-03-12_at_6.22.10_PM.png' | relative_url }}" alt="Screenshot_2025-03-12_at_6.22.10_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    

**Adjacent Sequence Interchange (a generalization of Adjacent Pairwise Interchange)**

> 問題變成從要下哪一條 chain  先加工，變成要在什麼時間點跳到別的 chain。也就是說不同 chain 可以切割，中斷換到下一條線加工

- Algorithm
    
   <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-03-15_at_10.31.20_PM.png' | relative_url }}" alt="Screenshot_2025-03-15_at_10.31.20_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
   1. max over $$l = 1,2,3,... k$$ 所有可以中斷的點
      1. 會 enumerate 不同的 chain
      2. 每個 chain 從 第一個 job 加第二個 job、第一個加第二個加第三個，以此類推找到最大的做下去
   2. 最大的就是 $$\sum w_j /\sum p_j$$ 就是中斷點
   3. LHS = $$\rho$$ $$\text{-factor}(1, 2,…, k)$$
- Proof
    
   假設這邊有三條不同的 chain 排列組成的大chain
    
   - $$\mathcal{S}^{\prime} = \{v, 1, 2, \cdots, l^*\}$$
   - $$\mathcal{S}^{\prime\prime} = \{ 1, 2, \cdots, l^*, v\}$$
   - $$\mathcal{S} = \{ 1, 2, \cdots, u, v, u+1, \cdots, l^* \}$$
   - 如果 $$\mathcal{S}$$ 比 $$\mathcal{S}^{\prime}$$ 好
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-06_at_7.31.49_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_7.31.49_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - 如果 $$\mathcal{S}$$ 比 $$\mathcal{S}^{\prime\prime}$$ 好
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-06_at_7.32.06_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_7.32.06_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - 根據定義 $$l$$
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-06_at_7.34.04_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_7.34.04_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - 所以不管在哪個條件下都會發現 其實 $$\mathcal{S^{\prime} }$$  或是 $$\mathcal{S^{\prime\prime} }$$ 都比 $$\mathcal{S}$$ 好

> **Note**
> **題目換成 $$1\vert r_j, prmp\vert  \sum W_jC_j$$**
>
> 如果 weight 都相同則可以用 Shortest Remaining Processing Time First (SRPT) 解決
>
> WSPT 修改成sum of weight / sum of remaining process time 也不適用

## Due Time Related Objectives

### **Maximum Lateness**

$$1\vert \vert L_{max}$$

- Lowest Cost Last 就可以解決了，因為 $$L_{max}$$ 是一種 non-decreasing function with time
- Earliest Due Date first (EDD)

$$1 \vert  r_j \vert  L_{max}$$

- Strongly NP-hard (3-partition problem reduce to this problem)
   - 3-partition problem
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-03-31_at_11.34.11_PM.png' | relative_url }}" alt="Screenshot_2025-03-31_at_11.34.11_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-03-16_at_9.41.49_PM.png' | relative_url }}" alt="Screenshot_2025-03-16_at_9.41.49_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
- Use Branch and Bound to search a best solution in a systematical way
   - Algorithm
      - Root是空排程
      - 每一層都會 job 的位置皆在上一層的後面 (第一層決定開頭)
      - Branch Rule
         - 如果此job在開始前，也就是 job release 前，就有 job release 後並完成工作，那該 job就不應該長出這個 job 要先做的分支，也就代表不應該先做
      - Bound Rule
            
         > **Note**
         > 如果 prmp-EDD 得到一個沒有任何插單的排程，此排程及為最佳解 (= lower bound)
            
         - 當前的排程可以隨意用一個排程法 e.g. EDD
         - 使用 prmp EDD 當作 lower bound (最好的狀況，因為在 $$1 \vert  r_j, prmp \vert  L_{max}$$ ，AKA沒有插單版本的 relaxation，可以用 prmp EDD 達到最佳解），此時就可以 argmin 留下，其他分支略過
   - 通常 Branch and Bound 會用來解 integer programming 的問題
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-06_at_8.35.12_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_8.35.12_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        

> **Note**
> if preemption is allowed then use **prmp-EDD** to get optimal solution

### Number of Tardy jobs

可以用來算on-time shipment (任務的成功率的感覺)

$$1\vert \vert \sum U_j$$  

- Algorithm (Forward Algorithm)
   1. 先把 job 根據 $$d_j$$ 由小到大排序
   2. 每個iteration按照順序取出一個job，直接加入排程結果J中
   3. 如果該job的完工時間 $$C_j > d_j$$，找出 $$J$$ 裡面 process time 最長的去掉 (其實很簡單，因為如果不去掉後面再排目標都不會變好，所以就是進一個出一個做比較久的，**讓整體能更早完工，看目標值會不會變好**)
   4. 下個iteration，會到第二步
   5. 剩下的一定會超過時間的job可以隨便排沒差

**More general case $$1 \vert  \vert $$ $$\sum W_jU_j$$**

NP-Hard Problem

當所有 due date 都相同時 = 背包大小，process time = item size， weight 就是 benefit of the item，這個問題就是一個背包問題

- Heuristically could use Weighted Shortest Process Time (WSPT)，但是效果不一定會好
- 反過來說如果這個背包問題的權重也就是benefit of item相同 (0,1 背包問題)，那就能用剛剛調的演算法

### Total Tardiness

$$1\vert \vert \sum{T_j}$$

這個指標很重要如果只用 number of tardy job，會有 job 可能要等超久

該指標為 ordinary sense of NP-hard，所以可以用 pseudo polynomial algorithm (based on DP) 來解。

- Preliminary Result
   1. 如果  $$p_j ≤ p_k,\ d_j ≤ d_k$$，則存在一 optimal sequence $$p_j$$ 排在 $$p_k$$ 前面
      - 這個定理刪掉很多排列，有點像是加上了一個 $$prec$$ 的限制 (Elimination Criteria, Dominance Result)
      - Proof
            
         <img src="{{ '/assets/notes/pos-ch3-single-machine-method/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
            
   2. Any sequence that is optimal for the second instance is also optimal for the first instance
        
      中間有一個 job $$k$$  的 $$d_j$$ 變成 $$\max{(d_k, C'_k)}$$
        
      <img src="{{ '/assets/notes/pos-ch3-single-machine-method/a9537f21-56b1-4833-89f6-2bb335b7598d.png' | relative_url }}" alt="a9537f21-56b1-4833-89f6-2bb335b7598d" loading="lazy" style="max-width: 100%; height: auto;" />
        
      - Proof
         - 對於 $$V''$$ 來說， job $$k$$ 在原本的排程 $$S'$$ 下 $$V''(S')$$ 排在哪裡都不會增加 Tardiness，因為超過 $$d_k$$ 的部分會有一個 max 使得 due date 變長，所以 $$V''$$ 會 ≤ $$V'$$
         - $$B_k$$ 代表的是是否 job $$k$$ 有因為 $$d_j$$ 換位子，選一個最小的，因為 $$V''(S'')$$是最佳解
         - 最後因為 $$S'$$ 是最佳解，所以 $$S''$$ 在 $$V$$ 也是最佳解
            
             
            
         <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-07_at_6.49.13_PM.png' | relative_url }}" alt="Screenshot_2025-04-07_at_6.49.13_PM" loading="lazy" style="max-width: 100%; height: auto;" />
            
   3. 在某個排程最佳解中，存在一 $$\delta$$ 使得 job $$k$$，會被排在 $$k+ \delta$$ 的位置
      - Proof
            
         <img src="{{ '/assets/notes/pos-ch3-single-machine-method/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
            
- Algorithm (Dynamic Programming $$O(n^3)$$)
   - 先根據 due date 小到大排好，並找出該 job set 最大的 process time job $$k$$，job $$k$$ 應該會排在比他 process time 小的 job 前面。
   - 運用下述遞迴演算法求解
      - $$J(j, l, k)$$ 代表從 $$\set{j, j+1, ... , l-1, l}$$，所有滿足 $$p < p_k$$ 的job
      - $$V(J(j, l, k), t)$$ 代表從時間 $$t$$ 開始排程 $$J$$，其最小的 total tardiness
      - $$C_k(\delta) = \sum_{j \le k + \delta} p_j$$
    
   $$ V(J(j, l, k), t) = \min_{\delta}\left(V(J(j, k^\prime + \delta, k^\prime), t) + \max(0, C_{k^\prime}(\delta)) + V(J(k^\prime + \delta+1, l, k^\prime), C_{k^\prime}(\delta))\right) $$
    
- Example
    
   <img src="{{ '/assets/notes/pos-ch3-single-machine-method/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Complexity
    
   <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-07_at_11.31.31_AM.png' | relative_url }}" alt="Screenshot_2025-04-07_at_11.31.31_AM" loading="lazy" style="max-width: 100%; height: auto;" />
    

### Total Weighted Tardiness

$$1\vert \vert \sum W_j T_j$$ NP-Hard in strong sense (3-partition)

- Algorithm
    
   Branch and Bound (decide the last job first)
    
   - Branch:
      - the job could find another job with following attribute can’t be at at the last position currently
         1. bigger process time
         2. bigger due date
         3. smaller weight
   - Bound: relaxation of the problem serving as lower bound
      - lower bound = transportation problem = $$1\vert prmp\vert \sum W_j T_j$$
      - Transportation Problem
         - 這代表每個工作可以被切分單位工作時間，並且運輸的成本在超過 $$d_j$$ 後開始產生
         - 設定指示變數標示 job 是否做完
         - 在 job 全部都有做完的情況下最下化成本
            
         <img src="{{ '/assets/notes/pos-ch3-single-machine-method/Screenshot_2025-04-07_at_11.01.27_AM.png' | relative_url }}" alt="Screenshot_2025-04-07_at_11.01.27_AM" loading="lazy" style="max-width: 100%; height: auto;" />
            
- Example
    
   <img src="{{ '/assets/notes/pos-ch3-single-machine-method/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

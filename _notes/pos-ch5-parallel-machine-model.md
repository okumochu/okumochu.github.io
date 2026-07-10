---
layout: page
title: "Ch 5 Parallel Machine Model"
category: lecture
course: "Production and Operations Scheduling"
order: 5
---

> $$P_m$$ Identical machines in parallel
 $$Q_m$$ Machines in parallel with different speeds (不管什麼 job 在同個 machine 有相同的速度 e.g. 機台老化)
 $$R_m$$ Unrelated machines in parallel (不同的 job 在不同的 machine 有不同的速度)

- Preemption can improve schedule even they arrive at same time
- Most of the schedule in parallel is non-delay **except unrelated machine**

## Minimize Makespan

$$P_2 \vert \vert  C_{max}$$ 是 NP-hard in ordinary sense， 因為等同於 2-partition

- 2-partition problem
    
   > In [number theory](https://en.wikipedia.org/wiki/Number_theory) and [computer science](https://en.wikipedia.org/wiki/Computer_science), the **partition problem**, or **number partitioning**,1(https://en.wikipedia.org/wiki/Partition_problem#cite_note-FOOTNOTEKorf1998-1) is the task of deciding whether a given [multiset](https://en.wikipedia.org/wiki/Multiset) *S* of positive integers can be [partitioned](https://en.wikipedia.org/wiki/Partition_of_a_set) into two subsets *S*1 and *S*2 such that the sum of the numbers in *S*1 equals the sum of the numbers in *S*2. Although the partition problem is [NP-complete](https://en.wikipedia.org/wiki/NP-complete), there is a [pseudo-polynomial time](https://en.wikipedia.org/wiki/Pseudo-polynomial_time) [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming) solution, and there are [heuristics](https://en.wikipedia.org/wiki/Heuristic) that solve the problem in many instances, either optimally or approximately. For this reason, it has been called "the easiest hard problem".2(https://en.wikipedia.org/wiki/Partition_problem#cite_note-hayes-2)3(https://en.wikipedia.org/wiki/Partition_problem#cite_note-FOOTNOTEMertens2006[httpsbooksgooglecombooksid4YD6AxV95zECpgPA125_125]-3)
   > 
   > 
   > However, it is quite different to the [3-partition problem](https://en.wikipedia.org/wiki/3-partition_problem): in that problem, the number of subsets is not fixed in advance – it should be |*S*|/3, where each subset must have exactly 3 elements. 3-partition is much harder than partition – it has no pseudo-polynomial time algorithm unless [**P = NP**](https://en.wikipedia.org/wiki/P_%3D_NP).5(https://en.wikipedia.org/wiki/Partition_problem#cite_note-Garey_&_Johnson-5)


**Longest Process Time (LPT)**

- Heuristic Rule，只要有機台是空的就把最長的 job 排上去
- Worst case (how well the heuristic is guaranteed to perform,  Upper Bound)
   - Proof
        
      找最小的作業數是因為，這樣就能保證運用 LPT 時，最後一個排上去的也會是最後結束的
        
      目標是要找到一個 最少 job 數，且能讓 LPT 排程的變差，OPT 變好。如果找到一個最後排上去的 job 不是最後結束，其實可以直接刪掉那個 job 完全不會影響 LPT 排程的 makespan，甚至 OPT 排程還會變好
        
      還沒排最後一個 job 的 makespan 上界其實就是要平均分配的狀況，再整理一下式子把最後一個 job 放進去。然後最佳解會等於或差於完全平衡 (總時數直接除以機台數) 的狀況
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/Screenshot_2025-04-04_at_5.51.14_PM.png' | relative_url }}" alt="Screenshot_2025-04-04_at_5.51.14_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - $$P_m \vert \vert  C_{max}$$
        
      $$ \frac{C_{max}(LPT)}{C_{max}(OPT)} \leq \frac{4}{3} - \frac{1}{3m} $$
        

## Project Planning

 **$$P_\infty\vert prec\vert C_{max}$$**

- Optimal Schedule
   - Schedule the job at time zero
   - Once the job has completed, start all jobs which predecessors has complete
- Given an optimal scheduling
   - If jobs start time can be delay without increasing makespan, called **slack jobs**
   - If it cannot postpone, called critical jobs, and they constitute **critical path**

**Critical Path Method (CPM)**

> Activity times are known with certainty and no variation allowed

- Example
   - 先 forward 再 backward
   - 紅色的就是 forward 可以算 earliness starting time。 predecessor  的 earliness starting time 加上 process time 取 max
   - 藍色的就是 backward 可以算 lateness starting time。 successor 的 earliness starting time 減去 process time 取 min
   - 綠色的就是 critical path，其他的是 slack job
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/Screenshot_2025-04-04_at_8.25.22_PM.png' | relative_url }}" alt="Screenshot_2025-04-04_at_8.25.22_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        

**Project Evaluation and Review Technique (PERT)**

> Activity times are treated as probabilistic


$$P_m\vert prec\vert C_{max}$$ 

> Strongly NP-Hard Problem


可以先從特例開始研究 $$P_m\vert p_j =1, tree\vert C_{max}$$

- Intree (at most one successor)
- Outtree (at most one predecessor)

**Critical Path Rule (CP Rule)**

- if all jobs process time = 1, intree or outtree，**CP rule** is optimal
    
   <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/Screenshot_2025-04-04_at_8.53.30_PM.png' | relative_url }}" alt="Screenshot_2025-04-04_at_8.53.30_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    

**Largest Number of Successors first (LNS)**

> 看所有 successor 總共有幾個 ，多的先

- if intree,  LNS = CP. So, LNS is optimal for intree
- 注意，不是直接後繼者，會將子孫全部一起納入

**Arbitrary Process Time**

- Revise the CP rule, and LNS from amount of node to amount of process time along the path
- two of them include itself，原本不用加入是因為大家 process time = 1 不影響

## Machine Eligibility

$$P_{m}\vert p_j = 1, M_j\vert C_{\max}$$

**The Least Flexible Job First (LFJ)**

選擇越少的job先處理

If $$M_j$$  is nested, then LFJ is optimal

- Nested means
    
   <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Example
    
   <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Makespan with Preemption

$$P_m\vert prmp\vert C_{\max}$$

> **Note**
> 可以看到 (4) 就是使得 $$C_{max} \ge p_j ,\forall j$$
> 加上這個條件就會迫使 makespan 至少大於最長的 process time

可以使用線性規劃模型 (LP)

$$ \begin{align}\min\quad&C_{\max}\\\text{s.t. }\ &\sum_{i=1}^{m}x_{ij} = p_j&j = 1, 2, \cdots, n\\&C_{\max} - \sum_{i=1}^{m}x_{ij} \ge 0&j = 1, 2, \cdots, n\\&C_{\max} - \sum_{j=1}^{n}x_{ij} \ge 0&i = 1, 2, \cdots, m\\&x_{ij} \ge 0&i = 1, 2, \cdots, m\\&&j = 1, 2, \cdots, n\end{align} $$

Lower Bound = $$C_{\max} \ge \max\left(p_1, \frac{\sum_{j = 1}^{n}p_j}{m}\right) = C_{\max}^*$$

- Note: $$p_1$$ 代表的是最長的工作
- Algorithm (Optimal)
   1. 假裝是單一機台
   2. 然後process time由大排到下
      1. 這樣可以確定不會同時間下 process 同時被兩個機台處理
   3. 平均切分成 m 份，丟給不同的 machine 做
   4. 每份就是 $$\frac{C_{max} }{m}$$

**Longest Remaining Process Time first (LRPT)** 

- 當時間切點為 間斷時 (e.g. 1, 2, 3, …) 可以得到最佳排程
   - 因為如果是連續的，兩個 job 就會無限交換
   - Example
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Machine with Different Speed

相較於 identical machine，多了一個速率 (e.g. 機台新舊) 

假設機器速率以降序排列 $$v_1 \ge v_2 \ge \cdots \ge v_m$$ 且 process time依舊由大到小排

Lower Bound 修正為 $$C_{\max} \ge \max\left(\frac{p_1}{v_1}, \frac{p_1 + p_2}{v_1 + v_2}, \cdots, \frac{\sum_{j=1}^{m-1}p_j}{\sum_{i=1}^{m-1}v_i}, \frac{\sum_{j=1}^{n}p_j}{\sum_{i=1}^{m}v_i}\right) = C_{\max}^*$$

- 總是使用最快的機台處理最長的 process time
- Proof
    
   <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

**Longest Remaining Processing Time on the Fastest Machine first (LRPT-FM)**

- Optimal for $$Q_m\vert prmp\vert C_{\max}$$ and $$Q_m\vert r_j, prmp\vert C_{\max}$$

## Total Completion Time

- $$P_{m}\vert \vert \sum C_{j}$$, $$Q_{m}\vert \vert \sum C_{j}$$ 這個兩個問題使用SPT rule 在仍然是最佳
   - 但是若是 weighted total completion time 就不是最佳的了，因為他是 NP-Hard  (找到一個反例就可以了)
- $$P_{m}\vert prec\vert \sum C_{j}$$ 是 strongly NP-Hard
- Critical Path Rule is optimal for $$P_{m}\vert p_j =1, outtree\vert \sum C_{j}$$，但是 intree 就不適用
- Least Flexible Job Rule is optimal when dealing with $$P_{m}\vert p_j =1, M_j\vert \sum C_{j}$$ ,and $$M_j$$ is nested
- $$R_{m}\vert \vert \sum C_{j}$$
   - Example
        
      每個 job 要選一個 machine 的 time slot 填進去，不同的 machine 有不同的速度
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
        
- $$Q_{m}\vert prmp\vert \sum C_{j}$$ 可以使用 **Shortest Remaining Process Time on Fastest Machine first (SRPT-FM)**
   - 把剩餘最短的時間的 process 總是分配給最快的機台處理

## Maximum Lateness

- $$P_{m}\vert \vert L_{\max}$$ 是一個 NP-Hard 的問題
- $$P_{m}\vert prmp\vert L_{\max}$$
   - Algorithm
      1. 令 $$z = 0$$
      2. 將 due date 改成 deadline，透過 $$d_j + z$$
      3. 從後面排回來，轉換一下問題變成 $$P_m \vert  r_j, prmp\vert C_{max}$$，deadline 就是 變成 arrival time
            
         <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
            
      4. 檢查 $$C_{max}$$ 的值是否 ≤ 原本的 deadline
      5. 如果 $$C_{max} > \text{max deadline}$$，把 $$z+1$$ 回到第二步重新跑一次
      6. 否則跳出迴圈 $$z = L_{max}$$
   - Example
        
      結果就是 $$C_{max} = 9 \le \text{max deadline} = 9$$
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
        
      <img src="{{ '/assets/notes/pos-ch5-parallel-machine-model/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />

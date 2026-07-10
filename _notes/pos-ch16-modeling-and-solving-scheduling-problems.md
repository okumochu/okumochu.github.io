---
layout: page
title: "Ch 16 Modeling and Solving Scheduling Problems in Practice"
category: lecture
course: "Production and Operations Scheduling"
order: 16
---

<img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/Screenshot_2025-05-30_at_5.42.14_PM.png' | relative_url }}" alt="Screenshot_2025-05-30_at_5.42.14_PM" loading="lazy" style="max-width: 100%; height: auto;" />

## Cyclic Scheduling of a Flow Line

- **Flexible Assembly Systems** (related to **buffer**)
    
   最後一句話的意思是說，Limited Buffer (buffer = $$n$$) 的問題可以想像成沒有 buffer 但中間有 $$n$$ 個 process time = 0 的 machine。之所以這樣轉換也是因為這樣就沒有 buffer size 的問題，所以整個問題會變得比較好解
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    
- 通常流水線工作都是 periodic 或是 cyclic。你可以想像當你有很多不同的 job 都要走過這些固定的 machine，上一個 job 做完下一個 job 又來了。如此反覆循環就換像是一個 circle (steady-state)

**Minimum Parts Sets (MPS)**

- 是指在所有不同的 product type 的個數  $$N_1, N_2,...~ N_l$$，取他們的最大公因數除上去 ，就會得到一個 vector。這就是 MPS
    
   $$ \bar{N} \;=\; \Bigl(\,\frac{\mathcal{N}_1}{q},\;\dots,\;\frac{\mathcal{N}_l}{q}\Bigr) $$
    

**Use Case**

- 當正在 build to stock 的時候，會對產品的生產量有一個比例上的安排 (product mix) ，並且在接下來的時間區段一直生產 (仍然保持比例)。此時就很適合 cyclic schedules，因為在達到總量前他會持續的生產相同比例的產品
   - e.g. Total = 30 Unit, A=10%, B = 40%, C=50%
- 並且透過 Assembly System 的概念轉換成 no buffer 的  $$F_m \vert block\vert  C_{max}$$ 的問題用 Profile Fitting 來解

> **Note**
> MPS的作用就是: 首先拿一個演算法，然後 input MPS 然後 output 一個時間區段的 schedule 然後就一直不斷地做那個 schedule 直到達到總生產量

## Scheduling of a Flexible Flow Line with Limited Buffers and
Bypass

想像一個有多個 stage 的平行機台問題 (Flexible Flow Shop)，除此之外 maximize throughput 的同時也要 minimize work-in-progress (WIP)

<img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

 ****

**Flexible Flow Line Loading (FFLL) Algorithm**

- **Phase 1: machine allocation**
   - 每個 stage 都會眾多 machine 中選一個出來做
   - 為了平衡使用的是 **Longest Process Time First (LPT)**
      - 只要有 machine idle，就把最長的 job 給他
- **Phase 2: sequencing**
   - 上一個 phase 主要做的事情就是 balance workload，至於在單一 machine 哪個 job 先做就是 sequencing
   - 為了解決短時間內在一個 arrive 很多 long process time 的 job，造成 queue up。使用 **Dynamic Balancing** **Heuristic** 來解決
      - 單一 machine (across stages) 的 workload =$$W_i \;=\; \sum_{j=1}^{n} p_{ij}$$
      - 該 system 的 workload (total workload) = $$W =\; \sum_{i=1}^{m} W_{i}$$
      - machine $$i$$ 在 job $$j$$ release 時的 workload，相較於該 machine 的 work load 有多少比例 = $$\alpha_{ij} \;=\; \sum_{k \in S_j} \frac{p_{ik} }{W_i}$$
         - $$S(j)$$ 代表一個 job set，這個 set 會 job $$j$$ (包含) 以前的全部的 job
      - 這個 **Dynamic Balancing** **Heuristic** 目的在於使 $$\alpha_{1j}, \alpha_{2j},…,\alpha_{mj}$$ 越靠近彼此越好
         - Measure the overload of machine $$i$$ due to job $$j$$ = $$o_{ij} = p_{ij} - p_j \frac{W_i}{W}$$
            - Ideal Workload  $$p_j \frac{W_i}{W}$$，也就是這個 job 在所有 machines 上總共需要花費的時間 $$\times$$ 這個 machine 平均來說負荷量多大
      - machine $$i$$ 在 job $$j$$ 以前的累積**超支的工作量**就是 = $$O_{ij} = \sum_{k \in S_j} o_{ik}$$
         - 就是目前在 machine $$i$$ 上的工作量加總 - 平均在 machine $$i$$ 的工作量
      - 用 Greedy Heuristic minimize 所有正的 $$O_{ij}$$ (只需要 minimized overload )
         - 每個 iteration 就是把上次選出來的 (加總最小的) column，加到其他人 column 上面然後取 max (added value, 0)
      - 最後得出的順序是不論什麼 machine 都是這個相對順序
- **Phase 3: timing of release**
   - 不是所有的 job 都需要 released at time = 0，因為會造成不必要的等待，所以目標就是在不影響 makespan 的狀況下盡量延後 release time
      - 在 machine allocation (LPT) 的時候就知道哪一個 machine 是 bottleneck，**就是圖上最長的那個**
      - 首先，把所有 job released 的時間越早越好
            
         > **Note**
         > 這邊的圖的虛線是代表下一個時期的 scheduling (一邊一個 MPS)
            
         <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
            
      - bottleneck machine 會固定，在該 machine 的**上游 machine delay 下游則是提早開始**
        
      <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
        

- Example
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
    
   這邊的 1, 2, 3 指的是不同 stage 的 process time，至於每個 stage 有多少 machine 可以看上面的圖片
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
    
   這邊只需要看那些 job 在第幾個 machine 做就好 (sequence 用 LPT 想就出來了)
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
    
   Wi 就是排在 machine i 上面的 process time 加總 (橫的加總)，然後 pk 就是該 job 總共需要的 process time (直的加總)
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />
    
   o matrix 畫出來就長這樣，然後取正的加總
    
   <img src="{{ '/assets/notes/pos-ch16-modeling-and-solving-scheduling-problems/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />

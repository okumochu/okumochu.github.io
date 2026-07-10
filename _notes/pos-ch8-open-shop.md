---
layout: page
title: "Ch 8 Open Shop"
category: lecture
course: "Production and Operations Scheduling"
order: 8
---

## Makespan without Preemption

$$O_2 \vert  \vert  C_{max}$$, $$O_2 \vert  prmp \vert  C_{max}$$

- **Lower Bound**:  $$C_{max} ≥ \max\{ {\sum^n_{j=1}p_{1j},\sum^n_{j=1}p_{2j} }\}$$ 每個機台上的 job 的 process time 加總

> **Note**
> 在這個問題下，出現 idle time 唯一可能性就是**最後一個 job** 正在做第一個 operation，但是另一個 machine 目前有空，所以就會等他一下
>
> 之所以是最後一個 job 的理由是因為是 non-delayed 的排程，所以如果發現要 idle 時就趕快換另一個不會 idle 的 job 就好了，所以除非是非不得已沒得換了 (最後一個) 不然不會 idle 

<img src="{{ '/assets/notes/pos-ch8-open-shop/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

- **Longest Alternate Processing Time First (LAPT) is optimal**
   - 當有 machine 是有空時，**其他 machine** remaining process time 最長的 job 排上去
   - 所以如果有一個 job 他已經在另一台 machine 上完成了，則他的順位就會變成最低 (=0)
   - 如果所有 job 的 priority 都是 0 ，誰先做就沒差了
- Proof
   - 不失一般性，先假設最長的 process time 是在第一個 machine 第 k 個 job ( $$p_{i,j}$$ ≤ $$p_{1,k}$$)
   - 這樣的話，此 job 就必須是 machine 2 的第一個 job。做完之後，該 job 在 machine 1 的 priority = 0 (postponed as much as possible)
   - 此時 idle time 可能會出現在 machine 1 或是 machine 2
      - **machine 2:** 就代表還有一個 job $$l$$ 必須先從在 machine 2 先做再做  machine 1。當這個 job 在 machine 2 做的時候， machine 1 現在是 job k 在做，很明顯 job k 一定會做到 job $$l$$ 做完，所以不會有 idle time
      - **machine 1:** 在這個情況下 job $$k$$ 一定是 machine 1 最後一個 job ，且 job k 在 machine 2 的 operation 還沒有結束。那此時的 makespan 就是 $$p_{1,k} + p_{2,k}$$ ，仍然 optimal

## Makespan with Preemption

$$O_m \vert  prmp \vert  C_{max}$$

- **Lower Bound**
    
    
   $C_{\max} \;\ge\;
   \max\Bigl\{
   \max_{j\in\{1,\ldots,n\} }\sum_{i=1}^m p_{ij}
   \;,\;
   \max_{i\in\{1,\ldots,m\} }\sum_{j=1}^n p_{ij}
   \Bigr\}.$
    
   - 直覺上看來就是 **column 的加總**或是**單一機台 row 的加總**
- Algorithm
   - Concept: 如果 schedule  result reach lower bound 代表最佳
   - row 
   $$i$$ or column 
   $$j$$ 如果加總 = lower bound 被稱為 tight ，否則就是 slack
      - 以 row 來說，就是某些 machine 可能從頭到尾都沒有 idle，一個 job 做完馬上下一個 job
      - 以 column 來說，就是某些 job 的加工跟加工之間沒有 idle，一個機器做完馬上就接下一個機器
   1. decrementing set rule
        
      這邊寫 at most 所以也可以不用選沒關係，但是**每個** tight 的 row 以及 column **都要且只能有一個** (所以 matrix 算完後，通常會先數一下)
        
      <img src="{{ '/assets/notes/pos-ch8-open-shop/Screenshot_2025-05-13_at_7.22.08_PM.png' | relative_url }}" alt="Screenshot_2025-05-13_at_7.22.08_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
   2. 先算出 $$C_{max}$$ ，很簡單就把每個 row column 都加起來取最大
   3. 把剛剛的 decrementing set 找出來 (每個 col 或是 每個 row 都有一個)
   4. **找出所有 delta 並且取最小**，這邊就分為兩種情況
      1. 如果是 tight row or column: delta = 該格的數值 $$p_{i,j}$$
      2. 否則就要把 slackness 加上去 = $$p_{i,j}$$ + Min ($$C_{max}$$ - sum of slack row, $$C_{max}$$ - sum of slack column)
   5. 取得delta 後，把所有剛剛選到的 decrementing set  排上根據 $$i$$, $$j$$ 排上甘特圖。最後在 Matrix 上減掉 delta
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Maximum Lateness without Preemption

$$O_m\vert r_j, p_{ij} = 1 \vert  L_{max}$$

- $$L_{max}$$ 的問題可以轉化為 $$C_{max}$$ with arrival time → deadline
   - each $$d_j =$$ $$d_j + L$$ , $$L_{max} ≤ L$$ ($$L$$ 其實會一直往上加，直到所有人都能在 deadline 前完成)
- Algorithm
   - 先找到 L 為多少，所以從 L = 0 開始試試看，慢慢再加大 L
   - 可以轉換成下面的圖的問題 (因為每個 process time = 1)
   - 每條從 job 連到 time 的都是單位為 1 的線，總共要連 m 條，**且連到同一個 time 下不能超過 m 條** (因為同一個 time slot 下只有能有 m 個 machine 在做事)
   - 以上的線連完之後，有可能不 feasible 因為同 job  不同 operation 在不同 time slot 上，但是在相同 machine 上。**所以我們需要修正的步驟**
   - 這就是一個 Bipartite graph (二分圖)。**可把圖切成兩半，各自的那半彼此都沒有連接**
   - 所以可以做成一個 | job | x $$t_{max}$$ 的矩陣，來表示是否有連接關係
   - 從左邊填到右邊，從上面填到下面。一直換顏色就可以了 (相鄰都不同顏色)
   - Maximum Lateness with Preemption
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Maximum Lateness with Preemption

$$O_2 \vert prmp\vert L_{max}$$

- 這個問題可以想像成 machine 1 或 machine 2 上面的工作時數加總**分別**是否能夠在 deadline 前完成。且總耗時的時間要小於 2 倍的 deadline - 兩機台都 idle 的時間
   - 所有條件都滿足時就代表有 feasible solution。也為了證明有滿足 $$z_j$$ 需要被遞迴算出
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
    

> **Note**
> 結果發現，存在一個演算法可以處理更 general 的問題 $$O_m \vert r_j, prmp\vert L_{max}$$

$$O_m \vert r_j, prmp\vert L_{max}$$

這個問題可以是一個 Linear Programming 問題，但是這個方法只能跟你說 $$L_{max}$$ 是多少 (是否能在 deadline 前排完)

- Algorithm
   - Definition
      - $$x_{ijk}$$ 就是在指當下的 interval $$k$$  裡要在 machine $$i$$ 做 job $$j$$ 做多久
      - 先把 release date 跟 deadline，取 distinct 一字由小到大排好
        
      <img src="{{ '/assets/notes/pos-ch8-open-shop/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - Constraint
      - 在限制式裡面的最後一條代表著，該 job 在特定 machine 上是否能在所有 interval 中的處理時間加總**小於**在**特定 machine 上總共需要的處理時間**
      - 這個限制式通常是要等於才對，因為工作必須要做完。因此如果限制式出來有**小於**的情況發生，就需要把 deadline += 1
        
      <img src="{{ '/assets/notes/pos-ch8-open-shop/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - 有了上述的東西之後，就可以拿去給 solver 解，**但沒辦法告訴你實際上在 time interval 中要如何分配**
   - 所以這時候就再把那個 interval 拿出來 轉換成一個 $$O_m \vert prmp\vert C_{max}$$ 的問題 (這邊的 m 就看有用到多少 machine)，然後再用之前的演算法解
- Example
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch8-open-shop/image-10.png' | relative_url }}" alt="image 10" loading="lazy" style="max-width: 100%; height: auto;" />

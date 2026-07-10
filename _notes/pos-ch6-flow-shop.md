---
layout: page
title: "Ch 6 Flow Shop"
category: lecture
course: "Production and Operations Scheduling"
order: 6
---

> 1. All jobs follow the same route (a sequence of operations)
2. Maybe limited buffer between neighbor machine (**blocking**)
3. The main objective of flow shop is $$C_{max}$$


> **Note**
> 這邊會開始出現 $$p_{ij}$$ 也就是不同的 job 在不同的 machine 上有不同的加工時間。前面探討的都是單一加工而已。 e.g. single machine, parallel machine (雖有平行，但是加工也只有一種)

**Unlimited Intermediate Storage**

> 指兩個operation之間的 buffer 是無限。通常這個在實務上是滿常見的只要生產的物品體積比較小就有可能滿足


## Flows Shops with Unlimited Intermediate Storage

有證明指出，在 flow shop 問題中，前兩個和後兩個加工的機台存在不用改變 job sequence 的最佳排程。也就是說 $$F_2\vert \vert C_{max}$$ and $$F_3\vert \vert C_{max}$$ 存在不用更換 job sequence (permutation, $$prmu$$) 的最佳排程 (一個沒有 permutation constraint 的排程問題會變得非常的困難)

對於一個 $$F_m\vert prmu\vert C_{max}$$ 問題，可以轉換成下列形式用來**算出 makespan**

- 算法 1 (recursive function)
    
   給定 job sequence $$j_1, j_2, …, j_n$$ ，$$C_{max}$$ 就可以透過以下方式算出來 
    
   max 是為了確保同一時間 machine 只會處理一個 job 不能重疊，如果上個 job 還沒做完，要等一下
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-21_at_4.22.22_PM.png' | relative_url }}" alt="Screenshot_2025-04-21_at_4.22.22_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
- 算法 2 (critical path on directed graph)
    
   可以把左邊這張圖畫成右邊的
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-21_at_4.37.53_PM.png' | relative_url }}" alt="Screenshot_2025-04-21_at_4.37.53_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-21_at_4.38.22_PM.png' | relative_url }}" alt="Screenshot_2025-04-21_at_4.38.22_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
   > **Note**
   > 右邊這張圖，其實就是指算法 1 的 max 邏輯
   > 算出右下角那個 job 的 earliest finish time 就是 $$C_{max}$$
    
   把右邊那張圖的甘特圖畫出來就會長下面這樣，因為 critical path 代表的是不能延後進行的 path (earliest starting time = latest starting time)，也就是說這個 path 上的 operation 都是緊密進行的一個接著一個
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-21_at_4.47.28_PM.png' | relative_url }}" alt="Screenshot_2025-04-21_at_4.47.28_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    

$$F_2\vert \vert C_{max}$$ **with** **Unlimited Intermediate Storage**

使用 **Johnson’s Rule** 得到**最佳解**

- 在 $$F_2$$ 下，所有的 job 有兩個加工 (機台) 要做可以分成
   - 第一個加工做比較快的 job → Set 1
   - 第一個加工做比較慢的 job → Set 2
- 排程的結果就是 Set 1 的 job **先**用第一個機台加工時間 SPT 排，Set 2 的 job **再**用第二個機台加工時間 LPT 排

> **Note**
> 主要的精神就是讓第二個機台不要 idle，所以第一個加工比較快的 job 趕快做一做就可以到讓第二個機台早點開工。這樣可能就上短下長 (甘特圖上)，此再用 Set 2 的 LPT 補回來
>
> note: 並不是只有這個 SPT(1), LPT(2) 能產生出最佳解

- Proof
   - 假設有兩個 job $$j$$ and $$k$$，**$$j$$ 在 $$k$$ 前面**，且這兩個 job 違反 Johnson’s Rule 的其中一個設定，證明遵守 Rule 的效果會大於或等於不遵守
      - 兩個不同 Set，$$j$$ 在 Set 2, $$k$$ 在 Set 1， (Set 1 應該要排在前面)
      - 兩個都在 Set 1，但是 $$j$$ 的第一個加工時間比 $$k$$ 長 (Not SPT)
      - 兩個都在 Set 2，但是 $$j$$ 的第二個加工時間比 $$k$$ 短 (Not LPT)
   - 現在有兩個排程 l → j → k → h (original) 和 l → k → j → h (interchanged)
      - $$C_{2, h}$$ = max ($$C_{2,k}$$, $$C_{1,h}$$) + $$p_{2, h}$$
      - new $$C_{2, h}$$ = max ( new $$C_{2, j}$$, new $$C_{1, h}$$) + $$p_{2, h}$$
      - 整個算式寫出來就會發現在上面三個任意一個情況下都滿足 new $$C_{2, h}$$ ≤ $$C_{2, h}$$
    

$$F_m\vert prmu\vert C_{max}$$

- 使用 Mixed Integer Programming (MIP)
   - 這個 $$I$$ 就是在指同一個 machine $$i$$ 下， 從當前 job 到下一個 job 需要 idle 多久
   - 這個 $$W$$ 就是在指同一個 job position $$k$$ 下，到下一個加工需要 wait 多久
    
   > **Note**
   > $$I$$ 和 $$W$$ 有彼此的關係化甘特圖出來就會知道
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-23_at_3.05.14_PM.png' | relative_url }}" alt="Screenshot_2025-04-23_at_3.05.14_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    
   > **Note**
   > Minimize makespan = Minimize the idle time on the last machine
    
- Slope Heuristic
    
   根據 slope index 由大到小來排序工作
    
   > **Note**
   > 越前面的加工應該要 process time 越短的先，越後面的加工應該要 process time 越長的越先。 e.g. 如果一 job 在最前面的加工中 process time 很大 乘上負的 weight後整個 slope index 就會很小使其 job 在順序上會往後排
    
   $$ \text{Slope Index}_j = - \sum^{m}_{i=1}{(m-(2i-1))p_{ij} } $$
    
   - Example
        
      <img src="{{ '/assets/notes/pos-ch6-flow-shop/ef8246d0-19b2-4716-abdd-2b15b05595ae.png' | relative_url }}" alt="ef8246d0-19b2-4716-abdd-2b15b05595ae" loading="lazy" style="max-width: 100%; height: auto;" />
        

## **Proportionate Flow Shop with Unlimited Intermediate Storage**

$$F_m\vert  p_{ij} = p_j\vert C_{max}$$ 指每個 job 在不同機台的 process time 都相同

- 這種退化的 Flow Shop 就能透過之前的演算法求出最佳解
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-23_at_5.04.03_PM.png' | relative_url }}" alt="Screenshot_2025-04-23_at_5.04.03_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Flow Shops with Limited Intermediate Storage

### Intermediate Storage = 0 **(No buffer)**

對於一個 $$F_m\vert block\vert C_{max}$$ 問題，可以轉換成下列形式用來**算出 makespan**

- Recursive Function
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Directed Graph
    
   Find the Critical Path
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

$$F_2\vert block\vert C_{max}$$

- 可以視為 TSP problem ($$C_{max}$$ 就是所有點經過的 cost 加總)
    
   想像成上下兩排同樣順序 job 的 operation 要做。如果**目前的 job上面的加工的結束時間比較大** 還是 **上一個 job 下面的的加工的結束時間比較大**，就要等他 (no buffer) 所以取 max。
    
   > **Note**
   > 同樣的 job 到下個加工的 cost = 0
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-25_at_10.48.42_AM.png' | relative_url }}" alt="Screenshot_2025-04-25_at_10.48.42_AM" loading="lazy" style="max-width: 100%; height: auto;" />
    

### Limited Intermediate Storage

$$F_m\vert block\vert C_{max}$$

如果 m ≥ 3 就是 strongly NP-hard

- Profile Fitting (Heuristic)
    
   $$D_{i,j}$$ = earliest departure time on machine $$i$$ job $$k$$
    
   有三個公式
    
   - **開頭機台**是要看我目前機台什麼時候做完，和那個正在佔用我的下一個加工機台的 job 什麼時候會走。取最大，因為 no buffer 所以如果占用機台的 job 不走也不能 release
   - **中間機台**是看上個加工機台我最快能走的時間 (上一步算出來的) 加上這個機台的加工時間，老樣子因為 no buffer 所以要看下個加工機台好了沒
   - **結尾機台**就是看最快能走時間，和目前這個機台加工完沒，取最大，再加上加工時間
    
   <img src="{{ '/assets/notes/pos-ch6-flow-shop/Screenshot_2025-04-24_at_9.21.36_PM.png' | relative_url }}" alt="Screenshot_2025-04-24_at_9.21.36_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    
   - 最後再把每個 job 的以下式子算出來挑出最當前最小的排他。因為這個式子代表當前排他 idle time 會最小
        
      $$
        
      \sum_{i=1}^{m} \left( D_{i,j_2} - D_{i,j_1} - p_{i,j_2} \right)
        
      $$
        
   - 下一個循環就會從剛剛排到第二個位置的 $$D_1$$ ~ $$D_m$$ 繼續

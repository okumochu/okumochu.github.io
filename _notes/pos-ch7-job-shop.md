---
layout: page
title: "Ch 7 Job Shop"
category: lecture
course: "Production and Operations Scheduling"
order: 7
---

<img src="{{ '/assets/notes/pos-ch7-job-shop/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

## Job shop with two machines

$$J_2 \vert \vert  C_{max}$$ 這個問題可以轉換成兩組 $$F_2 \vert \vert  C_{max}$$ 的問題

- 所有的 job 可以被分類成 J12 或 J21 (先做哪一個 machine)
- 兩組分別成為一個 Flow Shop 問題，然後用 Johnson’s Rule 來解
   - J12 → M1 as first machine
   - J21 → M2 as first machine
- 最後就會把 J12 的 sequence 先擺上去，再把 J21 的 sequence 擺上去
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Disjunctive programming

$$J_m\vert \vert C_{max}$$ 可以轉化成一個圖的問題 (graph)

- 總共有兩種 arc
   - 實線代表的是 conjunctive arcs 就是同樣的 job 要做 machine 的轉換
   - 虛線代表的是 disjunctive arcs 同樣的 machine 要進行下一個 job。特別的是這種連結會形成 Clique
- 接下來就是從每一個 pair of disjunctive arc，選出一個方向。然後，不能造成 circle (Feasible Solution )
    
   > **Note**
   > 這邊的不能造成 circle 也很好理解，就是少選一條 clique 裡面的 disjunctive arc 就好了。如果三條都選會變成這個 job 會被該 machine 重複做  
    
- 形成的有向圖，從 $$U$$ 到 $$V$$ 算出他的 critical path 就是 makespan

<img src="{{ '/assets/notes/pos-ch7-job-shop/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> 這種化為 Disjunctive Graph 來求最佳解的方式就叫做 Disjunctive Programming

- Disjunctive Programming
    
   Constraint Intro
    
   - routing 就是看有 job 有幾個 operation 就有幾個限制式，用來限制順序
   - 接下來就是用來算 $$C_{max}$$ 的
   - 用決定好的方向的 disjunctive arc 來限制順序，然後一個 machine 有多少個 job 就會有多少順序需要限制 (也代表選出了多少個方向)
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

## The shifting bottleneck algorithm

$$J_m \vert \vert  C_{max}$$ 在上面的區塊有介紹，但是那種問題用 B&B 爆開來解還是太花時間了，所以我們需要一個 heuristic 的方法來試試看解這個問題

主要的觀念是在每個 iteration 中找出目前是 bottleneck 的 machine

- Algorithm
   1. 一開始沒有任何 disjunctive arcs
        
      沒有 disjunctive arcs 反應在圖上的是原本要照 job 順序一個等一個，都變成了無限資源 (machine with infinite capacity) ，都不用等了可以平行做
        
   2. 每個 machine 都可以想成一個 $$1\vert r_j\vert L_{max}$$ (single machine) 的問題用 B&B 解
        
      > **Note**
      > 其實這邊提到的每個 machine 分開求解很好理解，因為每個 machine 的 disjunctive graph 會形成的就是各自的 clique，然後每個 clique 都當成一個 single machine 解他們 job 的順序 (disjunctive arc 的方向)
        
      $$r_j$$ = longest path from $$U$$ to $$(i,j)$$
        
      $$d_j$$ = 當前的 $$C_{max}$$ - longest path from $$(i,j)$$ to $$V$$ + $$p_{ij}$$ 
        
   3. 找出 $$L_{max}$$ 最大的那個，**因為這代表這個 machine 就是 bottleneck**
   4. 然後把該 machine 的結果 (job 的順序) 畫在圖上，更新當前的 $$C_{max}$$ 
   5. 最後用一樣 single machine 的方式檢測一下**是否需要 re-sequence 前面的所有 machine**
   6. 前往第二步，繼續排直到所有 machine 的順序都排好了
- Algorithm Textbook
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Example
   - $$M_0$$ 是指已經排好 job 順序的 machine set (by previous iteration)
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-10.png' | relative_url }}" alt="image 10" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-11.png' | relative_url }}" alt="image 11" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-12.png' | relative_url }}" alt="image 12" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-13.png' | relative_url }}" alt="image 13" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-14.png' | relative_url }}" alt="image 14" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-15.png' | relative_url }}" alt="image 15" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-16.png' | relative_url }}" alt="image 16" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-17.png' | relative_url }}" alt="image 17" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/pos-ch7-job-shop/image-18.png' | relative_url }}" alt="image 18" loading="lazy" style="max-width: 100%; height: auto;" />

---
layout: page
title: "Ch 14 General Purpose Procedures for Deterministic Scheduling"
category: lecture
course: "Production and Operations Scheduling"
order: 14
---

## Dispatching Rules

- Static rules (time independent): SPT, EDD
- Dynamic rules: Minimum Slack first (MS)
   - choose the smallest of max ($$d_j$$ - $$p_j$$ - 
   $$t$$, 0)

### Composite Dispatching Rules

會集合多種 dispatching rule 的想法把依照數值排序

**Apparent Tardiness Cost (ATC)** = WSPT rule + MS rule

$$ I_j(t)= \frac{w_j}{p_j}  \exp\!\Biggl(    -\frac{\max\bigl(d_j - p_j - t,\,0\bigr)}{K\,\bar p}  \Biggr) $$

Apparent Tardiness Cost (ATC) scheduling rule 

## Local Search

<img src="{{ '/assets/notes/pos-ch14-general-purpose-procedures/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

通常一個 Local Search 會在意四個議題

1. Representation 是否足夠好 (encoding, decoding)
2. 如何定義 neighborhood (解空間中有多少解是你現在的 neighborhood)
3. 如何 search neighborhood
4. 如何決定是否接受本次產生的結果
   1. probabilistic: simulated annealing
   2. deterministic: tabu search

### **Simulated Annealing**

- 模擬退火法是用下面式子 (的 $$\beta_k$$ 來退火。這個 $$\beta_k$$ 是一個單調遞減的數值
- 下面這個式子是接受比較差的 Scheduling Candidate 成為新的狀態的機率 。如果真的爛很多或是 $$\beta_k$$ 很大機率就會很低 (幫助收斂)
    
    

$$ P(S_k, S_c)= \exp\!\Bigl(\frac{G(S_k)-G(S_c)}{\beta_k}\Bigr) $$

- Algorithm
    
   <img src="{{ '/assets/notes/pos-ch14-general-purpose-procedures/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

### **Tabu Search**

- 會建立一個 list 裡面的 element 就代表著不能回去的狀態。如果 list 越長代表 exploitation 的能力就越好 (限制很多)
- 這樣做的主要目的是防止演算法立即返回到剛剛離開的解，演算法被迫探索其他的方向
- Algorithm
    
   <img src="{{ '/assets/notes/pos-ch14-general-purpose-procedures/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

### Genetic Algorithm

- 基因演算法講求的是適者生存 (適合不適合由 fitness function 決定)
- 每個階段 (block) 都有很多種不同的方式來實作

<img src="{{ '/assets/notes/pos-ch14-general-purpose-procedures/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

- Algorithm
   - Parameters
        
      <img src="{{ '/assets/notes/pos-ch14-general-purpose-procedures/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
        
   1. Encode
   2. Selection: Roulette wheel selection  $\mathrm{Prob}_{j}
   = \frac{\text{fitness}_{j} }
   {\sum_{i} \text{fitness}_{i} }$
   3. Cross Over: Linear Order Crossover (LOX)
        
      選一個 parent 的 sub sequence 直接貼到後代上，剩下的部分讓另一個 parent 根據順序填上去
        
      <img src="{{ '/assets/notes/pos-ch14-general-purpose-procedures/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
        
   4. Mutation: 0 to 1 or 1 to 0

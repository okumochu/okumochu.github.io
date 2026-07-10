---
layout: page
title: "Ch 15 More Advance General Purpose Procedures"
category: lecture
course: "Production and Operations Scheduling"
order: 15
---

## Beam Search **(束搜尋)**

做 Branch and Bound (B&B) 是一件非常耗時的事情。在節點數非常多的狀況下，每個節點都要評估好壞並計算出 lower bound 檢測是否有低於目前的最佳解 (該節點就會被丟掉) 是非常耗時的

- 不像 B&B 的目的是找出最佳解，Beam Search 找到足夠好的解就好了
- 透過定義一個 **Beam Width** $$B$$ 和一個 **Filter Width $$F$$**，算出所有的節點的做 **Crude prediction (** **heuristic approximation**) ****再取 $$F$$ 個做 **Thorough evaluation** (**bounded analysis**) **，**最後取 top $$B$$ 來 Branch，剩下的就丟掉了不管 lower bound
   - crude prediction 或是 thorough evaluation 都是需要自己去定義好的

## Decomposition Method

### Machine-based decomposition

e.g. shifting bottleneck 就是一種 machine-base decomposition

- 這種做法會 **schedule one machine at a time**，每次都從當前最難排的 machine 開始
   - 定義”難”有很多種不同的指標
- 找出最難排的 machine，找出該 machine 的 job sequence (成為一個 single machine 的 sub-problem)
   - 但是，這個 sub problem 也有可能不好解 (NP-hard) ，在不好解的況下也不知道這個解的品質有多大程度會影響 main problem 的解
   - 所以通常會有一個 control structure 使其每次在解完這輪的 machine 的時候會把前面的都 re-optimize。讓其在 main problem 上也可以有好的效果

### Job-based decomposition

- 這種作法會 **schedule one job at a time**，然後選目前 priority 最高的
   - 同樣的，定義 priority 有很多不同的指標
- 找出那個 job，並找到一個最好 (lowest cost) 的 machine 順序 (operation sequence) 然後插入這個順序
- 確保這個插入後的解仍然 feasible，若沒有任何解是 feasible，則適當的 reschedule 前面的 job
   - 在實務上通常就是直接把前面排的一個或多個 job 先撤出，最後再 re-insert

### Time-based decomposition

算是最通用的 decomposition，可以應用在任何 machine environment

- 通常會定義 time interval 的大小且只處理 arrive 在此 time interval 的 job；也可以定義成 fixed number of job 且一次只專注處理這麼多的 job
- 還有另一種可以更彈性更自然的 decompose
   - 找到一個 partitioning point time = $$t$$ ，使得在 $$t$$ 以前完成的 job sequence不會影響在 $$t$$ 之後完成的 job sequence
   - $$1\vert r_j\vert \sum{C_j}$$
      - $$V(t)$$ 定義為在 $$t$$ 之前還沒做的 job 還有多少 processing time 要做
      - 根據下圖，就可以看到 $$t$$ < 33 就是一個不錯的切割點
         - $$V(t)$$ 的值是透過 **$$t$$ 以前的工作的** **process time 加總 - $$t$$** 算出來的
        
      <img src="{{ '/assets/notes/pos-ch15-more-advance-general-purpose-procedures/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
      <img src="{{ '/assets/notes/pos-ch15-more-advance-general-purpose-procedures/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
        
   - $$1\vert \vert \sum{T_j}$$
      - 是透過把 $$d_j$$ 分群的方式切割問題
      - 下圖就會切成 job 1~4 和 job 5~9，分別解 $$1\vert \vert \sum{T_j}$$
            
         <img src="{{ '/assets/notes/pos-ch15-more-advance-general-purpose-procedures/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
            
- 當解完分割的問題之後，control structure 會做 post-processing procedure 就是**把中間的部分 re-optimize**。讓兩組子問題的解可以更融合
    
   <img src="{{ '/assets/notes/pos-ch15-more-advance-general-purpose-procedures/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

> **Note**
> control structure 其實就只是個概念，這個概念是想要利用 sub-problem 的解，並且輕量化的 re-optimize 讓這些 sub-problem 的解能夠在 main problem 有更好的表現

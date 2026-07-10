---
layout: page
title: "Ch 1 Traditional Machine Learning on Graphs"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 1
---

傳統機器學習像是 Logistic regression, Random forest, Neural networks 都是先在用一些資料訓練 (features of graphs)，然後再應用在不同的 graph 上。此時如何設計 feature 使其就算換到不同的 graph 也能完成下游任務就是非常重要的事情，也就是泛化的能力。以下用無向圖來說明

## Node-Level Features

<img src="{{ '/assets/notes/mlg-ch1-traditional-ml-on-graphs/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

以上面的任務為例，可以有以下的元素來描述這個 node 在 graph 中的結構上的表示

- **Node Degree:** 衡量有多少鄰居
- **Node Centralities $$c_v$$:** 衡量此 node 之於 graph 有多 “重要”
   - Eigenvector centrality: 當一個 node 被一群越重要的 node 連接代表他越重要
      - 這種蛋生雞雞生蛋的式子要收斂需要一些定理 Perron-Frobenius Theorem (PFT) ，其中 $$\mathbf{A}$$ 代表相鄰矩陣，且 $$\lambda$$ 是一個 normalization term
        
      $$ c_v := \frac{1}{\lambda} \sum_{u \in N(v)} c_u \iff \lambda \mathbf{c} = \mathbf{A}\mathbf{c} $$
        
   - Betweenness centrality (important for social networks):  ****node 出現在不同 node 到 node 的最短路徑越多次代表越重要 i.e. 社群網路中的中心 hub
        
      $$ c_v := \sum_{s \neq v \neq t} \frac{|\{\text{shortest paths between } s, t \text{ containing } v\}|}{|\{\text{shortest paths between } s, t\}|} $$
        
   - Closeness centrality: node 離其他 node 越近代表越重要
        
      $$ c_v := \frac{1}{\sum_{u \neq v} |\{\text{shortest paths between } u, v\}|} $$
        
- **Clustering Coefficients:** 衡量這個 node 鄰居連接的狀況
   - $$N(v)$$ 代表 v 的鄰居 (neighbor) / $$k_v$$ 代表 node $$v$$ 鄰居的個數 / Ego network: $${v} ∪ N (v)$$
   - 直觀去看就是鄰居彼此相連的邊數除上鄰居的個數的可以湊成三角形的組合數
   - 如果 = 1 代表其與鄰居是 fully connected (complete graph)，如果 = 0 代表 node $$v$$ 的鄰居彼此都是互相獨立 (必須通過 node $$v$$ 才能聯絡)
   - 如果以 ego network 的角度去看下面這個公式其實就是在分子是目前有幾個構成的三角形 (node $$v$$ 以及其他任意兩點) 除上最多能構成的三角形數
    
   $$ e_v := \frac{1}{\binom{k_v}{2} } |\{\text{edges among } N(v)\}| \in [0, 1] $$
    
   <img src="{{ '/assets/notes/mlg-ch1-traditional-ml-on-graphs/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

> **Note**
> 目前的建構出來的 feature 都是捕捉出 **local topology**，但是無法表現出這個 node 之於整張圖的結構

## Link-Level Features

- **Common Neighbor:** 直接計算這兩個節點有多少共同鄰居 (沒有算上自己)
    
   $$ |N(v_1) \cap N(v_2)| $$
    
- **Jaccard’s Coefficient:** 除上兩點鄰居的聯集的個數 = 上面正規化過的版本
    
   $$ \frac{|N(v_1) \cap N(v_2)|}{|N(v_1) \cup N(v_2)|} $$
    
- **Adamic-Adar Index:** 兩節點的共同鄰居如果連接到越多其他的節點 (popular) 分數越低
   - penalizes common neighbors that have a high degree
    
   $$ \sum_{u \in N(v_1) \cap N(v_2)} \frac{1}{k_u} $$
    

> **Note**
> 上面看到的這三個 link feature 時常都因為沒有共同的鄰居所以 = 0，且都是 local 1-hop information

- **Katz Index:** 不只計算一個 hop，計算到 infinite hop 彼此的鄰居是否相同，越遠 decay 越嚴重數值會小，directly friend 的增幅是最大的
   - $$\beta^i < 1 =$$ decay factor to prevent C from blowing up to + infinity
      - also providing the condition of invertible
   - $$A^i$$ counts the number of walks of length $$i$$ between two nodes
   - $$- I$$ is because the geometric series start from $$i = 0$$
      - $$1 + x + x^2 + x^3 + \dots = \frac{1}{1-x} \quad \text{(for } \vert x\vert  < 1 \text{)}$$
    
   $$ C :=\sum_{i=1}^{\infty} \beta^i A^i = (I - \beta A)^{-1} - I $$
    

## Graph Kernels

> The goal of graph kernels is to create a feature vector for the entire graph.
Kernel methods are widely used for traditional ML for graph-level prediction. Instead of designing feature vectors, we design kernels

- $$k(G, G') \in \mathbb{R}$$ : input two graphs project them into $$\mathbb{R}$$-dim
- $$K = [K(G, G')]_{G, G'}$$ must always be positive semi-definite
- There exists a feature representation $$\phi$$ such that $$K(G, G') = \phi(G) \phi(G')$$
   - 這個 $$\phi$$ 就是 mapping 原始資料 $$G$$ into high dim $$= G \rightarrow \phi(G)$$
   - Kernel Trick

通常會使用 bag-of-words 的方法來建構這個 graph-level feature

- **Graph-Level Graphlet features:** 數不同的 graphlets 各有幾個
   - 直觀來看兩個 graph kernel (投射到高維度內積) 的值越大越接近代表圖的結構越相似
   - 但是計算不同的 graphlets 各有幾個這件事情非常耗費困難 $$O(n^k)$$

> **Note**
> **這個部份著重在 graph structure 的 feature (how they connect together) 並且尚未考慮 node attributes**

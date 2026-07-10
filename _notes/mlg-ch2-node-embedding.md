---
layout: page
title: "Ch 2 Node Embedding"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 2
---

> **Representation learning** avoids the need of doing feature engineering every time. The goal
is to map individual nodes or an entire graph into vectors, or embeddings.


In node embeddings, we would like the embedding to have the following properties:

- 越相似的節點在 embedding space 中距離會越近
- Node embedding 需要包含 graph structure 的資訊 (i.e. topology)
- Potentially useful for downstream tasks
    
   <img src="{{ '/assets/notes/mlg-ch2-node-embedding/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    

接下來要介紹幾種能符合以上特性的 ecoding method

## Random Walk Embedding

隨機遊走的方法是一個 unsupervised 的方法也就是說演算法中不會需要用到 node labels or features。使用隨機遊走的好處在於 

1. **Expressivity:** 提供一種基於機率的相似度定義，也就是說兩個 node 的相似度跟這個 node 有多少機率可以走到另一個 node 有關
   1. 傳統的圖形定義有無連接的 binary variable 太過限制，這個方法把即使兩個節點沒有直接邊相連，只要透過隨機遊走能頻繁走到，它們就被視為相似
   2. 也就是說這個方法不只能提供 local structure information，也可以提供更遠的結構資訊
2. **Efficient:** 在訓練時不用完整的 traverse 整張圖

### Intuition

Random walk 就是在固定步數的限制下，不停的從 node $$u$$ 開始隨機遊走 (當然因為要蒐集資料每個點都會當作初始點 node $$u$$)，然後紀錄有經過的點，經過越多次越相似。然後接下來就是把這個會經過的機率算出來，接下來的目標就是把 node embedding (e.g. $$z_u$$ and $$z_v$$) 讓這樣個 node 的內積逼近這個”經過的機率”

### Loss function

<img src="{{ '/assets/notes/mlg-ch2-node-embedding/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

- 等號左邊就是收集的”經過的機率” 外面加上一個 log
- 最右邊這個藍色的機率就是模型要估計的，但是這個藍色的式子展開如下
   - 看到 softmax 的式子就會發現分母的部分加上原本上面最左邊的 summation 需要 iterate 所有兩兩 node $$= O(V^2)$$ 來做 normalization (i.e. 兩個 for 迴圈 )
   - 於是這邊提供一個快速的方法就是取 k 個，且通常者取樣的機率會根據 degree 的數量，degree 越大越容易被抽到
   - 接下來就是可以使用 stochastic gradient descent (SGD) 來 optimize 這個 vector $$z_u$$
    
   $$ P(v|z_u) := \log \frac{\exp(z_u \cdot z_v)}{\sum_{n \in V} \exp(z_u \cdot z_n)} \\\simeq \log \exp(z_u \cdot z_v) - \sum_{i=1}^{k} \log \exp(z_u \cdot z_{n_i}) $$
    

以上討論的這些叫做 **DeepWalk** 屬於 unbiased random walk 但是這個可能有點太限縮了，因為他是完全隨機的，它無法控制遊走的方向偏好。這導致它無法針對不同的任務需求，去側重捕捉圖形的**局部特徵** (Local View) 或是**全域結構** (Global View)

<img src="{{ '/assets/notes/mlg-ch2-node-embedding/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

### **node2vec**

這個方法 unbiased random walk 改成由以下兩個參數控制的遊走策略

- Return parameter $$p$$
   - 控制往回走的機率，使其更容易回到上一點 (BFS)
- In-out parameter $$q$$
   - 控制往外走的機率，使其更容易跑到比較遠的地方 (DFS)

<img src="{{ '/assets/notes/mlg-ch2-node-embedding/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> 雖然有更改的版本但是 random walk approach 總體來說是一個好方法

## Embedding Entire Graphs

有時候需要的 embedding 是整張圖而不是單一節點，而是針對整張圖 (e.g. graph prediction)。此時有以下幾種方法

1. **Sum over the node embeddings**: 利用已有的 node embedding 全部相加
    
   $$ z_G := \sum_{v \in G} z_v $$
    
2. **Virtual node:** 另設一個 global node (or hierarchical node)，並且跟圖 (或子圖) 上所有的點都有連接，該點的 node embedding 就是 graph embedding

## Relations to Matrix Factorization

剛剛提到的隨機遊走的 encode method，其實跟做矩陣分解 (matrix factorization) 有關node similarity 這件事其最簡單可以用相鄰矩陣來表示 (有沒有連接) 

- embedding matrix = $$d \times \vert V\vert $$ 的矩陣，每個欄位代表一個 node embedding
- 直觀上來其實 node embedding 其實就是要找一個滿足以下的 node embedding matrix，相鄰矩陣的值要與自己內積自己越靠近越好 (也就是 Z 乘上自己能夠還原成相鄰矩陣)
    
   $$ A \approx Z^\top Z $$
    
- 但是因為維度上  $$d \ll \vert V\vert $$ 所以通常相鄰矩陣 $$A$$ 是 full  or high rank，然而 $$Z^\top Z$$ 最多只有 $$d$$。所以其實不能直接真正用矩陣拆解做到這件事情來求出 $$Z$$，所以改為 minimize Frobenius norm 如下
    
   $$ \min_{Z} \|A - Z^\top Z\|_F \\ \|A - Z^\top Z\|_F = \sqrt{\sum_{i,j} (A_{ij} - (Z^\top Z)_{ij})^2} $$
    
   - $$Z^\top Z$$ 最多只有 $$d$$ 是因為 $$\text{rank}(Z^\top Z) \le \min(\text{rank}(Z^\top), \text{rank}(Z))$$
   - 剩下為什麼沒辦法直接求出來要用最小化來逼近就很直觀，因為你沒有辦法用一個 二維的素材來組合成一個三維的東西。另外 Frobenius norm 就是對矩陣每個位置算 MSE
- 舉例來說 DeepWalk encode method 在做以下矩陣拆解
   - (這邊我就沒有詳細花時間去了解下面的式子)
    
   <img src="{{ '/assets/notes/mlg-ch2-node-embedding/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
    

## Applications and Limitations

<img src="{{ '/assets/notes/mlg-ch2-node-embedding/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />

---
layout: page
title: "Ch 12 GNNs for Recommender Systems"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 12
---

<img src="{{ '/assets/notes/mlg-ch12-gnns-for-recommender-systems/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

主要的任務: 根據過往的 user-item interaction 預測 user 可能會喜歡的 new item

並且同常這個問題會 model 成一個 bipartite graphs

- 這是一個 link prediction 的問題 ( $$f(u,v) =$$ score)
    
   <img src="{{ '/assets/notes/mlg-ch12-gnns-for-recommender-systems/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

推薦系統的問題是 $$\vert V\vert $$ (user) 太多了，所以 check every pair 幾乎不可能

- 為了省資源，總歸來說會分成兩階段
   1. Candidate generation (fast): retrieve 1000 candidate item
   2. Ranking (slow, accurate): ranking each item with this user using  $$f(u,v)$$
   3. Get top-K item (10~100)
- Evaluation 會使用 recall at K
   - 有多少比例的 positive item (user $$u$$ would interact with in the future) 被預測到了
    
   $$ \text{Recall@}K := \frac{|P_u \cap R_u|}{|P_u|} $$
    

## **Embedding-Based Models and Loss Functions**

訓練下游任務有用的 embedding 來進行 input scoring function

上面提到的這個 recall@k 是沒有辦法微分的，所以以下有兩種 surrogate loss function

1. **Binary Loss** 
   - 這個的問題在於它強制把所有的在 training data positive edge 的分數拉高於 negative edge，但是問題在於每個 user 互相是 independent。下面的方法就切割開來了
        
      <img src="{{ '/assets/notes/mlg-ch12-gnns-for-recommender-systems/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
        
2. **Bayesian Personalized Ranking (BPR)**
    
   <img src="{{ '/assets/notes/mlg-ch12-gnns-for-recommender-systems/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

## **Neural Graph Collaborative Filtering (NGCF)**

傳統的 Conventional collaborative filtering  方法是用一個很淺層的 embedding 來做且沒有 node feature，score function 都是內積。用上了 message passing 的架構就能 capture k-hop structure 做到更好的 representation learning

<img src="{{ '/assets/notes/mlg-ch12-gnns-for-recommender-systems/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

## **LightGCN (simply version of NGCF)**

這個方法 argue that GNN 根本不需要這麼多參數量

- simplified representation format of adjacency & embedding matrices
- remove activation function (ReLU)
   - 這樣就只剩下 linear layer，就可以直接每層合起來做分析

## **PinSAGE (Pinterest's Industrial Model)**

這個模型會推薦 pins，並且這個 pins embedding 包含影像、文字、圖片資訊，並且節點數量是千萬級別。任務在於學習 pin embedding $$z$$ 使得相似的靠近不相似的遠離。以下有兩種訓練的小技巧

- **Shared Negatives:** Reusing the same set of negative item samples across all users in a mini-batch drastically cuts down computational overhead
   - 這個步驟是根據 loss function 的計算上的特性設計的
- **Hard Negatives:** Instead of using randomly selected items as negative examples (which are too "easy" for the model to distinguish), PinSAGE uses Personalized PageRank to find items that are somewhat related to the user but not actually interacted with
   - 這個步驟至關重要，因為如果負樣本亂選對模型來說會太簡單沒有學到難的東西 (e.g. contrasting a car with a dress)
   - 並且在這個步驟下有搭配課程學習，來越學越難

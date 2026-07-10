---
layout: page
title: "Ch 5 GNN Augmentation and Training"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 5
---

## Predictions with GNNs

目前已經學會如何用 NN 產出 embedding，接下來要介紹如何使用

1. **Node-level Prediction:** Directly make prediction using node embeddings using a linear prediction head
    
   $$ \hat{\mathbf{y} }_v := \text{Head}(\mathbf{h}_v^{(L)}) $$
    
2. **Edge-level Prediction:** Make prediction using pairs of node embeddings
   - 其中這邊的 head 可以是一個 linear layer、MLP、也可以是內積
        
      $$ \hat{\mathbf{y} }_{u,v} := \text{Head}(\mathbf{h}_u^{(L)}, \mathbf{h}_v^{(L)}) $$
        
3. **Graph-Level Prediction:** use global pooling (i.e. mean, max, sum) vector to predict, but for large graphs it loss information 
    
   $$ \hat{\mathbf{y} }_G := \text{Head}(\{\mathbf{h}_v^{(L)} : v \in V(G)\}) $$
    
   - Solution: Global Pooling Hierarchically  用兩個 GNN 來做抽取這個 global information
      - 先用 GNN-1 來進行 node embedding，再用 GNN-2 來做 cluster node 的任務
      - 所以使用方法就是 GNN-1 就是正常的做 representation learning 然後要抽取 global information 的時候就是用 GNN-2 來進行 cluster 直到 pooling 到一起
            
         <img src="{{ '/assets/notes/mlg-ch5-gnn-augmentation-and-training/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
            

## Training Graph Neural Networks

- Data source
   - Supervised Learning: 需要外部的資訊來辨別節點屬性
   - Unsupervised Learning / Self-Supervision: 其中是 Link prediction 因為就把原本的圖上的 Link 遮住就可以了
- How to split data
   - **Transductive Inference:**
      - **只有一張大圖**，訓練集、驗證集和測試集都在這同一張圖上，只是節點被分成了不同的組
         - 所以在訓練時就可以把節點跟邊通通放進去訓練，但是測試資料就是不獨立
      - 也就是說在訓練時，模型可以看到整張圖的結構 (包括測試節點的存在)，也可以利用節點之間的邊來進行訊息傳遞 (Message Passing)
            
         <img src="{{ '/assets/notes/mlg-ch5-gnn-augmentation-and-training/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
            
   - **Inductive Inference**
      - 訓練集和測試集是**完全不同、互不相干的圖**
         - 所以在訓練的時候必須把圖切開 into multi-graph
                
            <img src="{{ '/assets/notes/mlg-ch5-gnn-augmentation-and-training/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
                

### Example

- 以下的任務就是 link prediction 然後所以會先是有 training message edge 來自原本的圖要保留
- 接下來就是要 supervised 訓練的 edge 這個是要去除讓模型預測的
- 最後就是要看 test edge 有沒有要保留，transductive 的話就是要保留，並且預設的都是要保留來訓練

<img src="{{ '/assets/notes/mlg-ch5-gnn-augmentation-and-training/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

---
layout: page
title: "Ch 4 A General Perspective on Graph Neural Networks"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 4
---

設計一個 GNN 會包含以下五個元素

1. **Message** (node feature)
2. **Aggregation function:** 這個是區分不同的 GNN 的核心的關鍵
   1. e.g. Graph Convolutional Networks (GCN), GraphSAGE, and GAT (Graph Attention)
3. **Layer Connectivity:** Layer Connectivity refers to the topological connection between layers including skip connections.
4. **Graph Augmentation:** Feature augmentation, structure augmentation, etc.
5. **Learning Objective:** Supervised/Unsupervised, Node/Edge/Graph level objectives.

## Examples

- Graph Convolutional Networks (GCN)
    
   <img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    
- GraphSAGE
    
   <img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Graph Attention Networks (GAT)
    
   <img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

## GNN Layers in Practice

<img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

## Stacking GNN Layers

越多的 layer 能吸收到越遠的資訊但是同時也會有 over smoothing 的問題，這個問題會使得該層的 node embedding 的數值過於相似。有以下解法

1. 從根本下手一開始就不要疊這麼多層
   1. 作為補償提供多一點的模型複雜度來捕捉 feature 以及 decision head (MLP) for final task
   2. input the raw feature with MLP first, and use MLP for final task output
    
   <img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
    
2. 可以疊層數但是要提供多一些不同層數的資訊，不能只有前一層
   1. **Skip connection:** 概念是要在一個深層淺層都有的混合模型，使得較早期的層的資訊能保留到最後。當然也可以直接 Skip 到最後一層使最後一層要輸出前會有前面所有層的資訊
        
      <img src="{{ '/assets/notes/mlg-ch4-general-perspective-on-gnn/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Graph Manipulation in GNNs

原始的圖如果碰到以下狀況需要做調整

- **Feature Level:** 如果節點缺乏特徵就需要 data augmentation
   - Assign constant: 沒有資訊量
   - Assign ID: 沒有辦法應用在新的不同結構上的圖
- **Structure Level:** 圖的稠密度問題
   - **Sparse:** 資訊傳遞不夠遠
      - Add virtual link: 把 $$A^2$$ (2-hop) 能到達的點也直接連上變成 1-hop $$A=A+A^2$$
      - Add virtual node: 增加一個連接全圖所有節點的虛擬節點，大幅縮短任意兩點間的距離，幫助訊息傳遞
   - **Dense:** 計算量大
      - Neighbor Sampling: 在做 aggregation 的時候不要聚合所有鄰居

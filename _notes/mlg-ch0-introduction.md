---
layout: page
title: "Ch 0 Introduction"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 0
---

## Why Graph

- 現代 ML 的方法在做特徵抽取的時候會根據資料的結構給予 inductive bias 來建構模型，使模型能夠更有效率的捕捉資料的特性
   - e.g. 若是 sequential data ( text/speech ) 就會使用 RNN / image data (image) 就會使用 CNN 使模型能夠更快的適應
- 許多複雜的場景可以被畫成一個關聯圖 (relational graph) 於是乎可以使用 GNN 來進行特徵抽取
   - Graph Learning 是困難的因為其結構上來說只是表示兩個物件有關係，不像是 image data 有在於在歐式空間 (Euclidian Space) 的距離關係。並且 Graph Learning & Representation Learning 是有關係的，相似 node 投射到 d-dim 會有相似的 embedding
        
      <img src="{{ '/assets/notes/mlg-ch0-introduction/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
- Graph data 可以有以下幾種下游任務
   - Node Level Prediction: 根據 node 本身的 embedding 來預測
   - Edge/Link level prediction: 根據兩個 nodes’ embedding 來預測之間的關係
   - Graph Level Prediction: 根據子圖或整個圖來預測 e.g. traffic prediction / drug discovery
        
      <img src="{{ '/assets/notes/mlg-ch0-introduction/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Choice of Graph Embeddings

### Component of a graph

- Objects $$N$$: Nodes, vertices
- Interactions $$E$$: Edges, links
- Systems $$G(N, E)$$: Networks, graphs

很多時候有很多種不同的設計來表示這個資料結構，不同的設計會直接決定什麼樣子的資訊能夠被提取出來，以下有幾個設計上除了基本元素 (node, edge) 之外需要加以考量的特性

- Undirected/Directed edges
- Allow/Disallow self-loop
- Allow/Disallow multi-graphs (multiple edges between nodes)
- Heterogeneous Graphs (可以多種不同的 node type & edge type)
- Bipartite Graphs (e.g. Author-Paper graph)

> **Note**
> 現實世界中大多數的圖都是稀疏的 i.e. adjacency matrix is a sparse matrix with mostly 0’s.

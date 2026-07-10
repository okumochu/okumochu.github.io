---
layout: page
title: "Ch 3 Graph Neural Networks"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 3
---

<img src="{{ '/assets/notes/mlg-ch3-graph-neural-networks/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

原本很直覺的做法就是就是 GNN 應該要加入相鄰矩陣以及 node features 然後全部丟進 NN (MLP) 裡面，但是如果真的這樣做的話會有以下問題

1. $$O(\vert V\vert )$$ **parameters**
   1. 模型參數複雜度會隨著 node 變多跟著線性成長
2. **Graph size is inherently baked into the size of the neural network**
   1. input dim of MLP 會被訓練石的大小固定住，所以當 inference 有更多節點的圖就不能使用了
3. **Sensitive to node ordering**
   1. 圖中的節點是沒有特定順序的，但是若直接丟進 MLP 裡，排在前面的跟排在後面的是有區別的 i.e. 也就是說當 flatten 後的節點順序變了就會出問題 [node a, node b] & [node b, node a]

其中一個 解決方法是使用 **convolutional networks**，透過 sliding kernels 掃過在圖上的所有節點，這樣的好處是哪個點先掃後掃都無所謂並且參數量就是被 sliding kernel 決定而不是節點數量

所以現在要做的事情就是有一個 function $$f$$ map a graph $$G:=(A,X) = \mathbb{D}^d$$ 

接下來會定義兩種性質，某些圖的操作會包含這些特質

1. **permutation invariance:** 不論怎麼打亂輸入節點的順序都不會影響輸出結果
   1. 此特性的操作: 把所有節點 embedding 相加總和都一樣 (sum over node embedduibg)，用來總結全圖的資訊 e.g. pooling
2. **permutation equivariance:** 打亂了輸入節點的輸入，其打亂後的順序完全反應到輸出的順序
   1. 輸入的第 1 行變到了第 3 行，輸出的第 1 行也會跟著變到第 3 行
   2. 此特性的操作: 聚合鄰居的訊息。如果節點換了位置，聚合的訊息也會跟著一起換位置，並不會聚合到錯的節點上。這保證了輸出特徵仍然對應到正確的原始節點

<img src="{{ '/assets/notes/mlg-ch3-graph-neural-networks/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

## Graph Convolutional Networks

> A critical observation is that a node’s neighbourhood defines a **computation graph**.
Information can be passed along this computation graph to combine information from different parts of the graph.


**Neighborhood aggregation:** 這就是用來蒐集鄰居資料的方法，詳細的數學式如下

- GNN 中的 weight and bias 可以透過 SDG 來 optimize
- 細節上可以把運算轉化成矩陣的方式一次計算，不用用一個 for loop iterate all node
    
   <img src="{{ '/assets/notes/mlg-ch3-graph-neural-networks/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

## GNNs subsume CNNs

CNN 在結構計算上可以看作是一個 GNN 的特例

- CNN 是一個 grid graph，且每個節點的 degree 相同 (filter size 決定)
- 但是有一點要注意，傳統 GNN 對所有鄰居節點都是用**同樣的權重**做 message passing。但是CNN 對於鄰居 (每個相對位置 e.g. filter 左上角 or 右下角) 都有一個**專屬的權重**，這也是 CNN 有輸入順序的問題 e.g. 在左上的格子的資料跑到右下就會出錯

<img src="{{ '/assets/notes/mlg-ch3-graph-neural-networks/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

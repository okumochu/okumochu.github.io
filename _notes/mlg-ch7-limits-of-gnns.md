---
layout: page
title: "Ch 7 Limits of Graph Neural Networks"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 7
---

從上個章節就有提到一個好的 GNN 要做到的事情就是要考量到鄰居結構跟節點特徵 創建一個injective function ，所以根據以上定義一個好的 GNN 有以下特質

1. If two nodes have the **same** **neighborhood structure**, they must have the **same embedding**
2. If two nodes have **different** **neighborhood structure**, they must have **different embedding**

但是這兩個條件在現在的架構下還是會有些問題，也就是說**雖然不同的鄰居結構但是如果還是有可能有相同的 computation graph** 這樣的話計算上來說如果 node feature 相同不可能區分出來

<img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> 當然 computation graph 相同也就代表 node feature 要相同，但是此實在探討的是 GNN 僅憑這個鄰居結構能夠有多少表達能力。這個能力能為後續 node feature 不夠強或是有雜訊或是根本沒有節點特徵時時提供更好的 inductive bias。不然其實都指依靠 node feature，那也不用 GNN 了直接丟 MLP 就好

## Spectral Perspective of Message Passing

從下面的例子就可以發現，在高度對稱下 GNN 在現在計算 computation graph 的方式下，是沒有辦法很好的區分節點的位置 (position-aware) 或是 鄰居結構 (structure aware)，如下圖

- **Node level:** Different inputs with the same computational graph leads to GNN failure
   <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

- **Edge level:** Edge prediction tasks may fail since the nodes on the edges have identical computational graphs
   <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

- **Graph level:** Same overall computation graphs on different graphs lead to same prediction
   <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
    

從數學上來說可以分析  GIN（Graph Isomorphism Network）aggregation function，把他當作

- 原始的 GIN message passing 公式如下
    
   <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
    

- 先把 MLP 的一層分解出來看並寫成矩陣的形式，來看這層 MLP 是怎麼處理圖形的訊號
   - $$k = 0$$ 就是 self-loop 的那層 / $$k=1$$ 就是鄰居的聚合 / $$l$$ 就是 GNN layer
   - $$W_k$$ 就是 self-loop 跟鄰居有不同的權重
        
      <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
        

### Eigen-decomposition

- 對相鄰矩陣 $$A$$ 做 eigen decomposition = $$A = V \Lambda V^T$$
    
   > **Note**
   > 這裡對相鄰矩陣做 decomposition 可以看出滿多特性的
   >
   > <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
    
   <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />
    

> **Note**
> 主要的結論就是 數學上來說做完 eigen decomposition 後會得到結構特徵向量 $$v_n$$ 以及 node feature $$C$$。如果  $$C=1$$ 大家都相同的話，node embedding 就會是那些 eigen vector 沒有跟 $$1$$ 正交的，但是在對稱圖形中**許多特徵向量的數值總和為 0 (與 $$1$$ 正交)**

(待補) 詳細的數學推導因為數學不太完備，所以會等之後再回來補

# Solution: when computation graph are the same

## Feature-Augmentation: Structurally-Aware GNN

- **One hot encoding:** 給每個 node 一個 ID
   - 缺點: non scalability / cannot generalize to new nodes
- **Structural Feature Augmentation:** 計算節點結構上的特徵當作 feature
   - e.g. node degree / clustering coefficients (前面 graph structure feature 那邊的都可以用)
   - 或是可以用 adjacent matrix $$A^k$$ 來表示 cycle counts

> **Note**
> 透過引入這些結構特徵，GNN 的表達能力區分圖結構 (當 computation graph 相同時增加一些 feature 幫助判斷 e.g. circle counts)，但是如果我們遇到的問題是關於**位置**的 (e.g. 兩個節點結構完全一樣，但在圖的一左一右)

## Position-Aware GNN

> 
> 
> - **Structure-Aware Tasks**: Nodes are labeled by their structural roles in the graph
> - **Position-Aware Tasks**: Nodes are labeled by their position in the graph
> 
> <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />


GNN 通常在 position aware 的狀況下會比較容易失敗，因為 computation graph 根本一模一樣了

- 所以通常的解法就是加入一個座標系統，因此需要一個錨點 (anchor node) 基於以下定理
   - Bourgain’s Theorem, Informal
        
      <img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />
        
- 隨機抽取 anchor set $$O(\log^2n)$$ 計算目標節點 $$v$$ 最短距離，並且每次訓練 re-sample anchor set。在 inference 的時候就是也會先 sample 一組 anchor set 來用
   - **直接放入 node feature 中:** 比較簡單但是因為會 re-sample anchor set，雖然是同一個 set，但是 element 的擺放順序會影響到輸出
   - **先放入 NN 使其 Permutation Invariant:** 應該要有一個函數能將這些 set 直接轉換成距離的概念，這樣無論 anchor set 什麼順序擺放都不會改變這個 augmented feature 的值
      - 不是把這些 anchor set 當作是向量，因為向量兩個 element 交換會有影響 set 不會

## Identity-Aware GNN

(待補) 這個部分我實在不太確定意義為何 我先往下讀

<img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-10.png' | relative_url }}" alt="image 10" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-11.png' | relative_url }}" alt="image 11" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/mlg-ch7-limits-of-gnns/image-12.png' | relative_url }}" alt="image 12" loading="lazy" style="max-width: 100%; height: auto;" />

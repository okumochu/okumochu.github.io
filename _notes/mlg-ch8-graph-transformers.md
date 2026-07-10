---
layout: page
title: "Ch 8 Graph Transformers"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 8
---

本章節發現現在常用的 transformer 裡面的操作其實是可以看成 GNN 的操作。於是乎在很多下游任務中勝任的模型架構應該可以拿來幫助 GNN。

## A New Design Landscape for Graph Transformers

接下來會介紹如何把 graph 的資訊 (i.e. node feature / adjacency information / edge feature) 放進 transformer 做處理並滿足下面三種當初 transformer 的下面三種設計

1. **Tokenization:** 放 node feature 就可以了
2. **Positional encoding:** 這個比較困難，原本的 GNN 對於 positioned based task 同樣苦手
   - 可以用到之前提到的 random sample anchor set 計算距離
   - Laplacian Eigenvector Positional Encoding
      - $$L =$$ 兩節點有邊的節點上會是 -1 / 沒有邊會是 0 / 對角線則是保持 out degree number
            
         <img src="{{ '/assets/notes/mlg-ch8-graph-transformers/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
            
         D 就是度，用來顯示要多少向外連接的鄰居
            
      - 接下來把這個 $$L$$ 做奇異值分解 $$L = \Sigma \Lambda \Sigma^T$$，就取前 $$k$$ 個 eigen vector → node feature
         - 接著根據 eigen value 做排序，越大的代表 information 越 local 越小的代表越 global
         - 這可以很直觀的想像因為越 global information 每個 node 越相同
                 <img src="{{ '/assets/notes/mlg-ch8-graph-transformers/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
                
      - 前 k 個都放進模型裡就可以當作該 node 的座標 → positional encoding
         - $$\lambda_1$$ 對應的向量提供了第一維座標 (類似 X 軸)，$$\lambda_2$$ 對應的向量提供了二維座標
3. **Self attention:** 最後這個自注意力機制可以用來加入 edge feature
   - 原本的自注意力機制是假設全連接，這樣沒辦法表示出 GNN 是否有 connect
   - 比較直觀的做法就是加一個 bias $$a_{i,j} \mapsto a_{i,j} + c_{i,j}$$，並且如果有連接才會加這個 bias
      - 直接連接 $$c_{i,j} = w_e^T x_{i,j}$$
      - 間接連接 $$c_{i,j} = \sum_{k} w_{e_k}^T x_{e_k}$$ (把路徑上 edge feature 轉換後的全部加起來)

---

## Positional Encoding for Graph Transformers

剛剛講到 Laplacian eigenvector 還是不夠好， 因為 eigen vector $$v$$ 在計算時可能會正負號翻轉 (如下公式)。這樣會造成訓練不穩定因為 position embedding 的正負號意義不同

$$ Lv = \lambda v \iff L(-v) = \lambda (-v) $$

於是乎需要一個 sign invariant function 用來產生 sign invariant embedding 使其計算 eigen vector 後可以直接丟進去此 function 不用管正負號 (使用到以下定理)

- Theorem: 證明 sign invariant function 存在 ($$f(x) = f(-x)$$)，於是可以用 MLP 來逼近
    
   <img src="{{ '/assets/notes/mlg-ch8-graph-transformers/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

所以最終的流程就會變成

1. 計算 eigen vectors & eigen value 來排序取 $$k$$ 個
2. 然後把這些 eigen vector 拿過來通過 SignNet (sign invariant function)
    
   $$
   z_i  =\text{SignNet} (g(x_i) + g(-x_i))
    
   $$
    
3. Concatenate 原本的 node feature 以及剛剛的 positional embedding $$z$$，就完成了

---
layout: page
title: "Ch 9 Machine Learning with Heterogeneous Graphs"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 9
---

目前討論的 graph data 都假設只有一種 edge type，接下來會討論如何 model heterogeneous graph data，並且因為這種異質資訊能總是直接變成一種 feature (e.g. one hot encode) 所以何時需要 model 成這種異質圖 (不同 node type & edge type)

- Definition of heterogeneous graph
    
   $$N_r$$ 就是屬於該 type 的鄰居的集合
    
   <img src="{{ '/assets/notes/mlg-ch9-heterogeneous-graphs/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    
1. Different nodes/edges have different shapes of features
2. We know different relation types represent different types of interactions.

> **Note**
> 異質圖提供更強的 expressiveness，但是代價就是 computation overhead & required domain knowledge。**In practice choose the simpler homogeneous graph models first.**

<img src="{{ '/assets/notes/mlg-ch9-heterogeneous-graphs/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

## Relational GCN

可以先先從比較簡單的 GCN 開始延伸 (異質圖通常都是有向圖)

- 每個 type of relation 都有一組 weight 來做轉換
    
   <img src="{{ '/assets/notes/mlg-ch9-heterogeneous-graphs/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
    

- 原始的參數量 $$=\vert R\vert  \times d \times d$$ ，可能會造成 overfitting，可以考慮以下兩種 regularization
   1. **Block Diagonal Matrices** 
      - 把原本會互相交互做 feature extraction fully connected layer 變成每個 feature 切成 $$B$$ 組並且每一組平行輸入沒有交互作用，這樣能使參數量少了 $$B$$ 倍
      - Examples
         - **Group 1 (Indices 1, 2):** $$y_1$$ 的值只取決於 $$x_1$$ & $$x_2$$
         - **Group 2 (Indices 3, 4):** $$y_3$$ 的值只取決於 $$x_3$$ & $$x_4$$
                
            $$ \underbrace{\begin{bmatrix} y_1 \\ y_2 \\ y_3 \\ y_4 \end{bmatrix} }_{\text{Output } h'} = \underbrace{\begin{bmatrix} \mathbf{w_{11} } & \mathbf{w_{12} } & 0 & 0 \\ \mathbf{w_{21} } & \mathbf{w_{22} } & 0 & 0 \\ 0 & 0 & \mathbf{w_{33} } & \mathbf{w_{34} } \\ 0 & 0 & \mathbf{w_{43} } & \mathbf{w_{44} } \end{bmatrix} }_{\text{Block Diagonal } W} \times \underbrace{\begin{bmatrix} x_1 \\ x_2 \\ x_3 \\ x_4 \end{bmatrix} }_{\text{Input } h} $$
                
        
      $$ W_r := \text{diag}(W_{r,1}, \dots, W_{r,B}) = \begin{bmatrix} W_{r,1} & \cdots & 0 \\ \vdots & \ddots & \vdots \\ 0 & \cdots & W_{r,B} \end{bmatrix} $$
        
   2. **Basis/Dictionary Learning:**
      - 原本當有 100 種 relation 就會有 100 個 $$W_r$$，但是很多時候資料可能某個 relation 可能只出現一次 ，此時那個 relation 就會變得很脆弱。所以改成學習 basis & combination
      - 所以會定義 $$B$$ 組 basis $$=V_b$$ 以及對應的 combination weight $$=a_{r,b}$$
            
         $$ W_r := \sum_{b=1}^{B} a_{r,b} V_b $$
            
        

### Example

(待補) 這邊有一個小例子在講監督式學習 GNN 的完整流程，先跳過

## Design Space for Heterogeneous GNN

[Heterogeneous Graph Transformer]({{ '/notes/heterogeneous-graph-transformer/' | relative_url }}) 

> **Note**
> 異質圖的聚合會是將相同類型的使用 sum 聚合，不同類型的使用 concatenate 接起來。藉此來保有不同類型的語義

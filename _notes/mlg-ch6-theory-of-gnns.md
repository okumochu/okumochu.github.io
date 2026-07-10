---
layout: page
title: "Ch 6 Theory of Graph Neural Networks"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 6
---

<img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

- GNN 是透過 neighbor 相互連接的結構來區分不同 node，這個結構上的子圖可以被稱為 **Computation Graph =** 以自己為 root 向外擴張 k-hop (length = k) 的樹
   - 反過來說，兩個 node k-hop neighbor (computation graphs) 都相同，這兩個 node embedding 也要相同 e.g. 對稱圖形
        
      <img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
        
- 一個強大的 GNN 設計就是要能分辨不同 node，要做到這件事情就是在產生 embedding (aggregation) 的時候不能遺漏結構的資訊 = **表達力 expressiveness**
- 所以在 aggregation 的時候要**應該要用單射函數 (injective function)**
   - 單射函數能保證不同的輸入就會有不同的輸出 = one to one (一一對應)
        
      <img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Designing the Most Powerful GNN

根據上述的思路，最強大的 GNN 是要能區分不同結構的 node (embedding 要不同)

$$ \text{Aggregate}(\{x_u : u \in N(v)\}) $$

有很多種不同的 aggregation 的方式以 GCN、GraphSAGE 為例

- 假設有一個 multi-set，裡面的每個元素都代表 node embedding
- GCN 會做 mean pooling，GraphSAGE 做 max pooling
- 下面兩個 multi-sets aggregate 的結果會完全相同
    
   $$ \left\{ \begin{bmatrix} 1 \\ 0 \end{bmatrix}, \begin{bmatrix} 0 \\ 1 \end{bmatrix} \right\}, \quad \left\{ \begin{bmatrix} 1 \\ 0 \end{bmatrix}, \begin{bmatrix} 1 \\ 0 \end{bmatrix}, \begin{bmatrix} 0 \\ 1 \end{bmatrix}, \begin{bmatrix} 0 \\ 1 \end{bmatrix} \right\} $$
    

> **Note**
> 通常來說表達力 (是否能分辨不同的鄰居結構) 來說 **max < mean < sum** (加總比較接近 injective function，可以同時保留特徵的內容以及大小)
>
> <img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

順著這個邏輯想下去就知道，sum 還是不夠強。需要更接近 injective function 的 aggregation operation 被創造出來。於是就有以下定理 (Xu et al. ICLR 2019)

- Any injective multi-set function can be expressed as the following where $$\phi$$ and $$f$$, are non-linear function **i.e. MLP**
    
   $$ \\ \phi \left( \sum_{x \in S} f(x) \right) $$
    
   - Universal Approximation Theorem, Hornik et al., 1989
        
      <img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />
        
- **Graph Isomorphism Network (GIN)** 就是用以上的定理設計出來的
   - 直觀來看就是先用非線性函數轉換所有的 node feature 然後加總，再經過一個非線性轉換，就能保證資訊不丟失
        
      > **Note**
      > 用 MLP 來確保特徵轉換過程夠獨特，不會讓**不同的輸入算出同的輸出** (! injective)
        
      $$ \text{MLP}_\phi \left( \sum_{x \in S} \text{MLP}_f(x) \right) $$
        

### Discussion

<img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />

## **Graph Isomorphism Network (GIN)**

GIN 來自一個數學經典演算法 **Weisfeiler-Lehman (WL) Test / Color-Refinement algorithm**

- WL test 是用來判斷兩圖是不是相同 (isomorphic) 的演算法
   - 此演算法可以如果判斷出不相同則保證不相同，反之則不保證
   - 演算法 $$O(\vert E\vert )$$
      1. 初始化使每個節點都有自己的顏色
      2. 接下來每個點都蒐集自己 1-hop 鄰居的顏色
      3. 自己本身節點顏色加上 1-hop 鄰居的顏色會形成一個 multi-set，並且有相同元素的節點會在下一輪的時候變成相同顏色 (就是 hash 操作)
      4. 這個顏色更新會跑 k 輪，最後這個 $$c^{(k)}$$ 就會彙整 k-hop 的結構
    
   $$ c^{(k)}(v) := \text{hash table}(c^{(k-1)}(v), \{c^{(k-1)}(u) : u \in N(v)\}) $$
    
- GIN 本質上來說也是在做一樣的事情，只是這個演算法中的 hash function 改成用 MLP model
   - WL test 用的是 hash function 來區分顏色 (**discrete output: colors)**
   - GIN 用的是 MLP 來 mapping 該點是屬於什麼值 **(continuous output: node embedding)**
    
   $$ c^{(k)}(v) :=\mathsf{GINConv}\left(c^{(k)}(v), \left\{c^{(k)}(u) : u \in N(v)\right\}\right) := \mathsf{MLP}_{\phi} \left( (1 + \epsilon)\mathsf{MLP}_f(c^{(k)}(v)) + \sum_{u \in N(v)} \mathsf{MLP}_f(c^{(k)}(u)) \right) $$
    
   > **Note**
   > - GIN 的核心目標就是設計一個 aggregation function 能夠單射 (injective) 地將不同的圖結構 (multi-set) 投射到不同的 embedding
   >     - i.e. 無順序性 $$\{a, b\}$$ = $$\{b, a\}$$ / 可重複 $$\{a, a\}$$ ≠ $$\{a\}$$
   >     - 所以直觀來看這個 learnable $$\epsilon$$ 是為了能夠分出哪個是 node 本身哪個是鄰居，因為投射過後做 sum (if $$\epsilon = 0$$) 還是容易
   > - 證明上是先證明這個 aggregation function上面這個目標 function 存在，再利用 Universal Approximation Theorem 來證明 MLP 有這個能力能夠逼近任何 function
    
- 但是雖然 GIN 根據以上說明證明了**在 message passing 這個架構上 (computation graph)**理論表達能力最強，但是它存在一些限制。在機制上來說使用 GIN 就會不可行 (e.g. WL-1 limitation)，在應用時要特別避開，這個部分的內容下一章節會提到
- Advantage Summary
    
   <img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
    

# When things don’t go as planned

<img src="{{ '/assets/notes/mlg-ch6-theory-of-gnns/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />

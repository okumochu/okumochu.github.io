---
layout: page
title: "Heterogeneous Graph Transformer"
description: "Mutual attention, message passing, and relative temporal encoding, with a worked example."
category: research paper
order: 11
---

> Hu, Ziniu, et al. "Heterogeneous graph transformer." Proceedings of the web conference 2020. 2020.


## Motivation

- 很簡單就是之前所有的圖形都是設計成 homogeneous，但是以下都會不相同
   - node type (different feature)
   - edge type (different semantic meaning, including directions)
        
      <img src="{{ '/assets/notes/heterogeneous-graph-transformer/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
- 剩下就是因為過去的研究
   1. 為不同類型的節點或邊共享完全相同的參數 ( feature 轉換)，要麼為每種類型單獨設定權重，未能有效捕捉異質圖的複雜屬性 
        
      > **Note**
      > 傳統 GNN 在處理異質圖時，最大的困難在於如何處理不同類型節點之間的信息傳遞。如果使用相同的權重矩陣，會混淆不同類型的語意；如果為每種關係都設定一個獨立的矩陣，又會導致參數過多且難以泛化，特別是對於那些出現次數較少的關係 
        
   2. 沒有考慮動態性 (都是 static snapshot)，無法有效 handle dynamics structure dependency，本篇文章提出  temporal encoding technique
        
      > **Note**
      > 想像一下WWW這個會議節點。在我們的資料中，WWW@1994 意味著我們考慮的是第一屆 WWW，它更關注網路協議和 Web 基礎設施，而 WWW@2020 則意味著即將到來的 WWW，其研究主題擴展到社會分析、普適計算等領域。如果我們忽略時間，將這兩個不同時代的 WWW 視為同一個節點，就會丟失大量寶貴的上下文資訊
        
   3. 擴展性不足 (在數億 node 和 edge 規模下)，以及在大規模異質圖上的訓練 HG Sampling

## Methodology

- General GNN Structure: 會透過一個 Extraction 把與自己相鄰 node 的資訊抽取出來，然後所有相鄰的 node 都抽取出來後，接下來再跟自己做 Aggregation 成為自己 下一個 layer 的 embedding
    
   $$ H^l[t] \leftarrow \underset{\forall s \in N(t), \forall e \in E(s,t)}{\mathbf{Aggregate} } \left( \mathbf{Extract}(H^{l-1}[s]; H^{l-1}[t], e) \right) $$
    

> The goal of HGT is to aggregate information from source nodes to get a contextualized representation for target node $$t$$. Such process can be decomposed into three components: *Heterogeneous Mutual Attention, Heterogeneous Message Passing and Target-Specific Aggregation.*


<img src="{{ '/assets/notes/heterogeneous-graph-transformer/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

### Heterogeneous Mutual Attention

- 這個部分就是 attention 裡面 query, key 概念的呈現，但是這邊要注意的是每種 source 或是 target 都有自己專屬投影到 query 或 key 的線性矩陣 (保持語義一致)
- 中間的是一個根據不同邊關係的矩陣，這是用來捕捉不同邊關係的關係
- 最後最右邊有一個 trainable $$\mu$$，這個是用來衡量這三元關係的重要程度 (source, edge, target)
- 以下就是每個 attention head 都是這樣計算，然後最後就是把所有 head concat 在一起最後通過 softmax 標準化 (基本 attention 操作)

$$ \text{ATT-head}^i(s, e, t) = \left( K^i(s) W_{\phi(e)}^{\text{ATT} } Q^i(t)^T \right) \cdot \frac{\mu(\tau(s), \phi(e), \tau(t))}{\sqrt{d} } $$

### Heterogeneous Message Passing & Aggregation

- 這個部分就是 attention 裡面 value 的展現 (這邊叫做 message)， 記算起來比跟剛剛類似，先進行投影後再呈上一個補捉邊關係的矩陣 $$W_{\phi(e)}$$ (很明顯的上標 MSG, ATT 是不同的語義上的轉換)

$$ \text{MSG-head}^i(s, e, t) = \text{M-Linear}^i_{\tau(s)}\left(H^{(l-1)}[s]\right) W_{\phi(e)}^{\text{MSG} } $$

### Target-Specific Aggregation

- Aggregate 的動作，在計算出每個鄰居的注意力和訊息後，我們需要將它們 Aggregate 到目標節點上 。我們直接使用注意力向量作為權重，對相應的訊息向量進行加權平均，得到更新後的向量  (下面這個式子就是用 attention 加權平均)
    
   $$ \tilde{H}^{(l)}[t] = \sum_{\forall s \in N(t)} \left( \text{Attention}_{\text{HGT} }(s, e, t) \cdot \text{Message}_{\text{HGT} }(s, e, t) \right) $$
    
- 最後需要將這個聚合後的向量映射回目標節點 (先過 activation function 後線性轉換一下)，再加上自己是一層的 embedding 就全部結束了
    
   $$ H^{(l)}[t] = \text{A-Linear}_{\tau(t)}\left(\sigma\left(\tilde{H}^{(l)}[t]\right)\right) + H^{(l-1)}[t] $$
    

> **Note**
> **參數共享發生在節點類型的層級，而模型的特異性則保留在邊類型的層級**
>
> 讓我們透過一個具體的例子來拆解這個過程。假設我們有兩個非常相似的元關係：
>
> - **關係 A**: `<作者 (Author), 是第一作者 (is_first_author_of), 論文 (Paper)>`
> - **關係 B**: `<作者 (Author), 是共同作者 (is_co_author_of), 論文 (Paper)>`
>
> 如果是一個沒有參數共享的 naïve 模型，它會為關係 A 和關係 B 分別學習兩套完全獨立的參數。這不僅參數數量驚人，而且模型無法將從關係 A 學到的關於作者和論文的知識應用到關係 B

### Relative Temporal Encoding (RTE)

- 主要是為了把相同 node，但是不同的時間發生的區別出來，藉由讓模型知道誰相對來說比較早比較晚就可以區別了
   - 對於一組 source, target，首先現計算時間差距 $$\Delta T(t, s) = T(t) - T(s)$$
   - 接下來輸入到 sin 或是 cos 函數裡面來 encoding
        
      > **Note**
      > 這個編碼器必須滿足兩個條件：
      >
      > 1. **獨特性與可區分性:** 不同的時間差 `ΔT` 應該對應到不同的向量
      >     1. 如果只有 sin，sin(0) = sin(π) = 0
      > 2. **平滑性與連續性:** 相似的時間差 `ΔT` 應該對應到相似的向量
      >
      > 這樣才能夠增加 generalization 能力，因為模型可能沒有看過 $$\Delta T = 1000$$ 的狀況，但是**一個好的 encoding 就能使模型內插和外推** 
        
      - 所以我們選擇的是兩個 sin 和 cos，並且要定義 一個向量
            
         $$ \begin{align*} \text{Base}(\Delta T(t,s), 2i) &= \sin\left(\Delta T_{t,s} / 10000^{\frac{2i}{d} }\right) \\ \text{Base}(\Delta T(t,s), 2i + 1) &= \cos\left(\Delta T_{t,s} / 10000^{\frac{2i+1}{d} }\right) \\ \text{RTE}(\Delta T(t,s)) &= \text{T-Linear}\left(\text{Base}(\Delta T_{t,s})\right) \end{align*} $$
            
         - 所以會先決定的是 d，決定後就會有 d/2 個組不同的 sin/cos pair，組成 d 維度的編碼，每一對的頻率是由分母 $$P_i =10000^{\frac{2i}{d} }$$ 決定。**d 越大維度越大，整個頻率顆粒度就越細**
         - 舉例來說如果 d = 6，就會有 `[sin(ΔT/P₀), cos(ΔT/P₀), sin(ΔT/P₁), cos(ΔT/P₁), sin(ΔT/P₂), cos(ΔT/P₂)]`
                
                
            | ΔT | `sin(ΔT/1)` | `cos(ΔT/1)` | `sin(ΔT/21.5)` | `cos(ΔT/21.5)` | `sin(ΔT/464)` | `cos(ΔT/464)` |
            | --- | --- | --- | --- | --- | --- | --- |
            | **0** | 0.00 | 1.00 | 0.00 | 1.00 | 0.00 | 1.00 |
            | **1** | 0.84 | 0.54 | 0.046 | 0.99 | 0.002 | 1.00 |
            | **2** | 0.91 | -0.42 | 0.093 | 0.99 | 0.004 | 1.00 |
            | **10** | -0.54 | -0.84 | 0.45 | 0.90 | 0.021 | 0.99 |
            | **500** | -0.99 | -0.12 | -0.59 | -0.81 | 0.88 | 0.47 |
   - 最後通過線性轉換加到 source node的原始表示上
        
      $$ \hat{H}^{(l-1)}[s] = H^{(l-1)}[s] + \text{RTE}\left(\Delta T(t, s)\right) $$
        
      <img src="{{ '/assets/notes/heterogeneous-graph-transformer/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />
        

### TRAINING Technique

<img src="{{ '/assets/notes/heterogeneous-graph-transformer/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

### HGSampling

- 傳統的圖神經網路 (GNN) 訓練方法是全批次 (full-batch) 的，需要一次性計算所有 node representation，這在大規模圖中是不可行的 。雖然現有的採樣方法 (如 GraphSage、FastGCN) 可以解決這個問題，但它們在異質圖上會產生節點類型極度不平衡的子圖，因為不同類型節點的數量和度分佈 (degree distribution) 可能差異巨大
- 為了解決此問題，提出了 **HGSampling (Heterogeneous Mini-Batch Graph Sampling)** 演算法
   - HGSampling 主要的目的就是不要一次訓練整個網路，而是透過合理的 sample nodes，然後用那些 nodes (sub-graph) 來進行訓練。並且這些 sample 出來的 node 有以下特性
      - **維持類型平衡:** 確保採樣出的子圖中，各種類型的節點和邊的數量大致相當
      - **保持子圖密集:** 盡可能使採樣的子圖保持緊密，以減少資訊損失和採樣變異

> **Note**
> mini-batch 就是只每次 training iteration 的時候就**只會挑選一小部份的 node 來更新和預測 (下游任務)**，因此每次的 iteration 都要重新做一次 HGSampling 為這個新的 mini-batch 生成專屬於它的 `OS` 和 `Â`  (sub-graph 來訓練)

- 下面就是這個演算法
   1. 先給定一個進 Output Node Set (`OS`)，然後把 Sampling Node Set (`NS`) ← `OS`。最後不同節點類型 $$\tau$$ 都會有一個 budget ($$B[\tau]$$)，並且接下來都是從這個 `NS` 來操作
   2. **Initialize Budget:** 遍歷 `NS` 中的每個節點 $$t$$，將其所有鄰居節點 (也就是所有可能的 source) 加入對應類型的預算 $$B$$ 中 
      1. 在加入時，會計算每個 source 的關聯重要度 (後續 sample 的時候會用到)
         1. 可以想像每個 target $$t$$ 都有自己的投票，當這個 target $$t$$ 有連接到很多人，這個票就會被分散，所以每個 source 拿到的票數就會比較少
         2. 另外，如果這個 source 被很多 target 連接到，這代表這個 source 很重要。整體上來說票數也會變高。
   3. **Iterative Sampling 重複 L 層 (每次都會擴增 OS)**
      1. 對於每個類型的 source
         1. **進行抽樣:** 根據 budget 調整抽取的機率，越重要的機率越高，抽取 $$n$$ 個點
         2. 把這些抽到的點加入輸出的集合 `OS` 然後把他們的鄰居 (source node) 都加到 Budget 中 (一樣加入的時候會根據連接的數量給一個值，用來表示關聯性)
         3. 最後就是把這些抽到的點移出 budget (本來這些點就是從某個 target 放進去的 budget 的)
   4. 最後就可以用 `OS` 和 `Â` ，進行訓練

<img src="{{ '/assets/notes/heterogeneous-graph-transformer/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/heterogeneous-graph-transformer/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />

### Inductive Timestamp Assignment

- Inductive Timestamp Assignment 的做法就是讓沒有 timestamp 的 plain nodes 繼承 event node 的 timestamp
   - Plain Nodes: 這類節點沒有內建的、固定的時間戳 。它們的屬性或扮演的角色會隨著時間演進 e.g. 一個會議 node 可以在不同的時間點舉辦，並且其研究的主題就可以隨之變化
   - Event Nodes: ****這類節點本身就帶有明確且固定的時間屬性 。最典型的例子就是 **Paper**，一篇論文有其確定的發表日期，這個時間戳是它內在的屬性
- 主要目的為
   1. 之前講到的 RTE 會需要 source node $$T$$ 和 target node $$T$$，這樣才能夠被計算
      1. 舉例來說，在處理一篇2019年發表的論文時，與它相連的 WWW 節點就被臨時賦予 2019年這個時間戳。這使得模型能夠理解，我們正在討論的是 2019年的WWW，而不是 1994年的WWW
   2. 為了分清楚不同時間下但是相同類型的 node。不分配時間戳，模型會為 WWW 這個節點學到一個涵蓋了它從1994年到2020年所有特性的、非常模糊且平均的 embedding

<img src="{{ '/assets/notes/heterogeneous-graph-transformer/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />

## Experiment

- Summarize by Gemini
    
   ### 第五章：評估 (EVALUATION)
    
   本章節的結構如下：
    
   1. **5.1 Web-Scale Datasets**：介紹實驗所使用的資料集
   2. **5.2 Experimental Setup**：說明實驗任務、評估指標、比較基準模型及實作細節
   3. **5.3 Experimental Results**：呈現並分析實驗結果
   4. **5.4 Case Study**：透過案例探討模型捕捉圖譜動態性的能力
   5. **5.5 Visualize Meta Relation Attention**：視覺化分析模型自動學習元路徑的能力
    
   ---
    
   ### 5.1 Web-Scale Datasets (網路規模資料集)
    
   為了全面評估模型在真實世界中的應用，作者使用了
    
   **開放學術圖譜 (Open Academic Graph, OAG)** 作為實驗基礎 
    
   - **OAG 主資料集**：
      - **規模**：這是目前公開可用的最大異質學術資料集，包含超過 1.78 億個節點和 22.36 億條邊
      - **時間跨度**：資料涵蓋了從 1900 年到 2019 年的學術文獻
      - **節點類型**：共有五種節點：'Paper' (論文), 'Author' (作者), 'Field' (領域), 'Venue' (場所，如會議/期刊), 和 'Institute' (機構)
      - **邊類型細化**：為了捕捉更豐富的語意，作者對邊關係進行了細化：
         - 'Paper-Field' 關係根據領域的層級（L0 至 L5）進行區分
         - 作者的順序（第一作者、最後一作者、其他）被區分
         - 場所的類型（期刊、會議、預印本）也被區分
         - 除了對稱的自環邊，所有其他關係類型都存在一個反向關係`φ⁻¹`
   - **領域特定子圖**：
      - 為了測試模型的泛化能力，作者還從 OAG 中構建了兩個特定領域的子圖：**Computer Science (CS)** 和 **Medicine (Med)**
      - 這兩個子圖的規模依然龐大，包含數千萬節點和數億條邊，比現有研究中常用的 DBLP 或 Pubmed 等資料集大至少一個數量級
    
   ---
    
   ### 5.2 Experimental Setup (實驗設定)
    
   - 四大下游任務：
      1. **Paper-Field (L1) Prediction**：預測論文屬於哪個 L1 級別的領域（節點分類任務）
      2. **Paper-Field (L2) Prediction**：預測論文屬於哪個 L2 級別的領域（節點分類任務）
      3. **Paper-Venue Prediction**：預測論文發表在哪個場所（節點分類任務）
      4. **Author Disambiguation**：作者名稱消歧，判斷同名作者發表的論文是否屬於同一個人（連結預測任務）模型首先透過 GNN 得到論文和作者的表示，然後使用一個神經張量網路 (Neural Tensor Network) 來預測它們之間存在連結的機率
   - **資料劃分**：採用基於時間的劃分方式，確保評估的現實性
      - **訓練集**：2015 年以前發表的論文
      - **驗證集**：2015 年至 2016 年發表的論文
      - **測試集**：2016 年至 2019 年發表的論文
   - **評估指標**：使用兩個廣泛應用於排序任務的指標：**NDCG** (Normalized Discounted Cumulative Gain) 和 **MRR** (Mean Reciprocal Rank) 。所有實驗均重複 5 次並報告平均值與標準差
   - **基準模型 (Baselines)** ：
      - **同質圖 GNN**：
         - **GCN**：簡單地對鄰居嵌入進行平均
         - **GAT**：對鄰居使用多頭注意力機制
      - **異質圖 GNN**：
         - **RGCN**：為每種關係三元組（source type, edge type, target type）維護一個獨立的權重矩陣
         - **HetGNN**：為不同類型的節點採用不同的 Bi-LSTM 來聚合鄰居資訊
         - **HAN**：設計了基於元路徑的層級式注意力網路
   - **消融研究 (Ablation Study)**
      - 為了驗證 HGT 兩個核心組件的有效性：**異質權重參數化 (Heter)** 和 **相對時間編碼 (RTE)**，作者設計了 HGT 的不同變體進行比較
   - **輸入特徵 (Input Features)**：
      - 由於 HGT 不要求所有節點特徵屬於同一分佈，作者為不同類型的節點設計了最合適的特徵
      - **論文**：使用預訓練的 XLNet 處理論文標題，並透過注意力加權平均得到標題表示
      - **作者**：其發表論文表示的平均值
      - **領域、場所、機構**：使用 metapath2vec 模型預訓練得到它們的嵌入
      - **公平比較**：為所有模型（包括基準模型）增加了一個**適配層 (adaptation layer)**，透過線性投影將不同類型的輸入特徵映射到同一個分佈空間
    
   ---
    
   ### 5.3 Experimental Results (實驗結果)
    
   - **主要發現**：
      - 在所有資料集的所有任務上，無論是 NDCG 還是 MRR 指標，HGT 的表現都
            
         **顯著且穩定地優於**所有基準模型
            
      - 以 OAG 資料集上的 Paper-Field (L1) 任務為例，HGT 的 NDCG 指標相比基準模型提升了 15-19%，MRR 指標提升了 18-21%
      - 與最強的基準模型 HAN 相比，HGT 在 CS、Med 和 OAG 資料集上的平均相對 NDCG 提升分別為 11%、10% 和 8% 36。總體平均性能提升約 20%
   - **效率與參數**：
      - HGT 的參數數量比 RGCN、HetGNN 和 HAN 等其他異質圖模型
            
         **更少**，且每個批次的訓練時間相當
            
      - 這證明了透過元關係三元組來參數化權重矩陣的策略，能夠在消耗更少資源的同時達到更好的泛化能力
   - **消融研究結果**：
      - 移除**異質權重參數化 (-Heter)** 會導致模型性能下降約 4%
      - 移除**相對時間編碼 (-RTE)** 會導致模型性能下降約 2%
      - 這兩個結果證明了 HGT 的兩個核心組件對於提升模型性能都至關重要
    
   ---
    
   ### 5.4 Case Study (案例探討：會議主題的時序演化)
    
   本案例旨在展示 RTE 元件捕捉圖譜動態性的能力
    
   - **方法**：選取 100 個高引用的 CS 會議，為它們分別賦予 2000、2010、2020 三個時間戳，然後利用訓練好的 HGT 模型得到它們在不同時間點的嵌入表示，並找出最相似的 Top-5 會議
   - **發現**：會議之間的關聯性隨時間發生了顯著變化
      - **WWW**：在 2000 年，它與資料庫 (SIGMOD, VLDB) 和網路 (NSDI) 關係密切；到 2020 年，則更接近數據挖掘和資訊檢索領域 (KDD, SIGIR, WSDM)
      - **KDD**：從 2000 年與傳統資料庫會議 (SIGMOD, ICDE) 的強關聯，演變到 2020 年與機器學習 (NeurIPS)、Web (WWW)、AI (AAAI) 等多個主題產生關聯
      - **NeurIPS**：模型成功捕捉到它在 2020 年與新興的頂級深度學習會議 ICLR 的緊密關聯
    
   | 會議 | 時間 | Top-5 最相似會議 |
   | --- | --- | --- |
   | **WWW** | 2000 | SIGMOD, VLDB, NSDI, GLOBECOM, SIGIR |
   |  | 2020 | KDD, GLOBECOM, SIGIR, WSDM, SIGMOD |
   | **KDD** | 2000 | SIGMOD, ICDE, ICDM, CIKM, VLDB  |
   |  | 2020 | NeurIPS, SIGMOD, WWW, AAAI, EMNLP |
   | **NeurIPS** | 2000 | ICCV, ICML, ECCV, AAAI, CVPR |
   |  | 2020 | ICML, CVPR, ICLR, ICCV, ACL |
    
   ---
    
   ### 5.5 Visualize Meta Relation Attention (視覺化元關係注意力)
    
   本節旨在證明 HGT
    
   **無需手動設計元路徑 (meta-path)，即可自動學習出重要的資訊傳遞路徑**
    
   - **方法**：將 HGT 前兩層中擁有最高注意力值的元關係序列視覺化
   - **發現**：
      - 為了計算**論文 (Paper)** 節點的表示，模型發現最重要的 2-hop 路徑等價於傳統意義上的元路徑，例如：
         - `Paper - Venue - Paper` (PVP)
         - `Paper - Field - Paper` (PFP)
         - `Institute - Author - Paper` (IAP)
      - 這些路徑的重要性是模型從資料中**自動學到**的，而非人工預設
      - 視覺化結果證明，HGT 能夠隱式地為特定下游任務學習和構建最重要的元路徑

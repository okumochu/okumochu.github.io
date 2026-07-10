---
layout: page
title: "Flexible Job Shop Scheduling via Dual Attention Network-Based Reinforcement Learning"
description: "Dual-attention method with the math spelled out, plus a note on a missing ablation."
category: research paper
order: 13
---

[GitHub - wrqccc/FJSP-DRL: This repository is the official implementation of the paper “Flexible Job Shop Scheduling via Dual Attention Network Based Reinforcement Learning”. IEEE Transactions on Neural Networks and Learning Systems, 2023.](https://github.com/wrqccc/FJSP-DRL?tab=readme-ov-file)

> Wang, Runqing, et al. "Flexible job shop scheduling via dual attention network-based reinforcement learning." IEEE Transactions on Neural Networks and Learning Systems 35.3 (2023): 3091-3102.


## Motivation

- 需要有一個更好 state representation 使得已經完成的 operation (不必要的資訊) 能夠被捨去。如果不捨去整張圖會變的很 dense 影響你
- 現有的研究只把 operation-operation 或是 operation-machine relationships，沒有考慮 machine-machine 之間的關係
    
   > The relationship between machines can be viewed as the **competition for remaining unscheduled operations** which is crucial for discriminating high-priority machines

    

### Contribution

1. A tight state representation for describing operations and machines in FJSP that is minimal and sufficient for downstream decision-making with the state space decreasing as scheduling proceeds.
2. A Dual Attention Network (DAN) consisting of several operation and machine message attention blocks for deep feature extraction of operations and machines.
3. A size-agnostic DRL-based approach with (markedly) improved performance and generalization capability compared to conventional PDRs and the state-of-the-art
DRL method.

## Methodology

<img src="{{ '/assets/notes/daniel-dual-attention/Screenshot_2025-09-02_at_11.10.53_AM.png' | relative_url }}" alt="Screenshot_2025-09-02_at_11.10.53_AM" loading="lazy" style="max-width: 100%; height: auto;" />

### State

主要這個部分就是會想辦法去除不需要的資訊量，記錄的資訊分成三部分

1. relevant operation node: dim $$\in \R^8$$
2. relevant machine node: dim $$\in \R^{10}$$
3. compatible operation machine pair: dim $$\in \R^{10}$$

過去單純在用 disjunctive arc implement GAT 的時候，都會把各種 relation (conjunctive, disjunctive arc) 都用同一個 attention 來 model

### Action & Reward

- Action Space 會根據 unscheduled operation 的數量改變 i.e. ($$O$$, $$M$$) pair。然後會 input 環境的 state + 每個 pair 的 embedding → 輸出一個 scaler (等同於打分數)。最後把所有 pair 的分數過 softmax 就可以選一個 action 出來了
   - 當然這種方式就會有兩種選 action 的方法，第一種是每次都選分數最大的，另一種就是每次都用 sample 然後求解到底 100 次然後找出最好的
    
   <img src="{{ '/assets/notes/daniel-dual-attention/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
    
- Reward 就是 potential-base reward shaping (telescope sum) **but with lower bound**
   - 本篇使用 lower bound 的方式來記算，每次會計算 lower bound 的變化，但是原本是計算 current maskspan 的變化
   - 原本的公式如果 discount factor = 1 的話， cumulative reward 就會是 $$-C_{max}$$
   - 但是本篇論文的 cumulative reward 就會是  $$\text{lower bound}-C_{max}$$
      - 這邊的 lower bound 是指初始還沒有排的 lower bound，當然這個 lower bound 會隨著你做出錯誤的選擇之後慢慢變大 (變差)
      - 初始還沒有排的 lower bound 計算方式就是把每個 job 的每一個 operation 的 minimum process time 加總，然後加總最大的那個 job 的那個值就是初始的 lower bound

### Dual Attention Network

主要就是把複雜的關係**每一層**用以下兩個 block 來表示，總共有 $$L$$ 層

1. **Operation** **message attention block:** 用來表示 operation feature 和彼此 precedence 的關係
   1. 其實計算方式跟 GAT 非常相似，只是關係沒有這麼複雜，只考慮了前後的關係
   2. 首先就是算出前後 operation 的 attention score: $$e$$ ， $$\vert p-j\vert $$ 是為了只抓前後一個 operation)
      1. 其中 $$h$$ 就是代表 operation 的 node feature，$$W$$ 和 $$A$$ 都是 trainable parameters。以下操作也很直觀，就是把兩個 node 接起來各自先 linear transform 之後再一起 linear transform 最後過 activation function
        
      $$ e_{i, j, p} = \text{LeakyReLU}(a^T[Wh_{O_{i,j} } || (Wh_{O_{i,p} })]) \quad \text{for all } |p-j| \le 1 $$
        
   3. 有了這些 attention score 之後，會先 mask 掉已經做完的 operation 的資訊。使得某些邊的 weight = 0，該 operation node 的資訊就會被捨棄
   4. 最後通過一個 softmax 正規化後做 aggregation，也就是正規化後的 attention score 做加權平均，就得到該 operation 的下一層的 presentation
    
> **Note**
> 這邊的每一個 operation 都會通過 $$L$$ 層的計算，用的都是同一組參數來取 feature。所以最後每一組 operation node 都會有 $$L$$ 個 embedding，看是要做 concat 還是 average，這樣 L 層幾乎能抓到整個 $$J$$ 的 feature
    
2. **Machine** **message attention block:** 用來表示彼此 machine 動態競爭的關係 (兩台機器如果都能加工某個相同的待排程作業，它們之間就存在競爭關係)
   1. 他有一個自己定義的競爭強度，這個強度是一個向量直接將 machine $$k$$ 和 machine $$q$$ 目前可以加工的 operation node**s** feature 相加 $$= c_{kq}$$  (本研究說可以顯示出競爭的強度)
        
      $$ u_{kq} = \text{LeakyReLU}(\vec{b}^{\top}[(Z^{1}\vec{h}_{M_k})||(Z^{1}\vec{h}_{M_q})||(Z^{2}c_{kq})]) $$
        
   2. 之後也是所有跟 machine $$k$$ 有競爭關係的都會過一個 softmax 做正規化，然後做加權平均得到下一層 machine $$k$$ 的embedding
    
> **Note**
> 這邊有一個不錯的點就是 $$c_{kk}$$ 其實可以表示 machine processing capacity (可以以及不能加工的 operation 相同)，並且在沒有可以加工的 operation 的時候會變成 0 使得資訊能被捨棄
    
3. **Integration and Output:** 
   - **Multi-Head Attention:** operation and machine blocks 都採用了多頭注意力機制。不同頭的輸出會在除了最後一層都是被 concatenate (e.g. 4 個 attention head 每個輸出 8 維，最後這個 layer 的這個 embedding 就是輸出 32 維)，到了最後一層 $$L$$ 才是平均起來 (輸出 8 維)
   - **Pooling:** 在通過 L 層 DAN 後，模型會對所有相關作業的最終特徵和所有相關機台的最終特徵分別進行 pooling (averge)，然後將兩者 concat 起來，形成一個代表當前排程狀態的全域特徵 (Global Features)**。**這個全域特徵會與單個作業、機台的特徵一起被送入下游的決策網路
    
    

<img src="{{ '/assets/notes/daniel-dual-attention/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

這圖真的畫得不是這麼的好懂…

> **Note**
> 這邊主要設計的點使得邊數 = $$O(3\vert O\vert +\vert M^2\vert )$$，相對於原本的 $$O(\vert O^2\vert )$$ 簡潔很多。因為通常 Operation 的數量是遠大於 Machine 的數量

如果以演算法來看的話，其實大家都是像下面一樣這樣做。我可以發力的地方就是下面這個換 training data (environment) 的這個地方，可以變得更有效率

<img src="{{ '/assets/notes/daniel-dual-attention/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

## Experiment

> **Note**
> 這邊有一個比較奇妙的點在於，他 claim 說他捨棄冗餘 operation 資訊 或是 增加 machine 競爭資訊可以做得更好，但是事實上他沒有做這個相關的 ablation study，所以整體效果 outperform 的歸因就有待商榷

- Summary by Gemini
    
   ### **第四部分：實驗驗證 (Experiments)**
    
   為了全面評估我們提出的 DANIEL 演算法，我們將其與多種基準方法進行了比較，這些方法包括了流行的優先派工法則（PDRs）、Google OR-Tools 精確求解器、一種先進的遺傳演算法，以及目前最先進的 DRL 方法。我們的評估涵蓋了排程效能、泛化能力和計算效率，並使用了合成生成的實例和廣泛使用的公開基準測試集。
    
   **A. 數據集 (Datasets)**
    
   我們設計了兩類具有不同分佈的合成數據，以檢驗 DANIEL 的學習與泛化效能。
    
   1. **SD1 數據集**：此數據集改編自文獻 [31]，其特點是允許不同工作的工序數量不同。
   2. **SD2 數據集**：這個數據集中的工序加工時間具有更寬的隨機範圍。具體來說，一個
        
      n×m 的實例中，每個工作有 m 個工序，其可選機器數量和加工時間分別從 U(1,m) 和 U(1,99) 的均勻分佈中採樣得出。
        
    
   我們在四種較小規模（10×5,20×5,15×10,20×10）的實例上進行訓練，並在包含這四種規模以及額外兩種更大規模（30×10,40×10）的未見過的測試數據上進行評估 7。此外，為了探索模型處理跨分佈任務的能力，我們還在四組公開基準測試集（包括 Brandimarte 的 mk1-10 和 Lawrence 的 la1-40）上進行了評估。
    
   **B. 基準方法與評估指標 (Baselines and Performance Metrics)**
    
   我們的比較對象涵蓋了不同類型的演算法：
    
   - **優先派工法則 (PDRs)**：我們選擇了四種被證明效能良好的 PDRs，包括 FIFO、MOPNR、SPT 和 MWKR。
   - **高效能演算法**：我們使用 Google OR-Tools 作為精確求解器的參考，設定了1800秒的時間限制 10。同時，我們也引用了一種兩階段遺傳演算法（2SGA）的結果作為參考。
   - **最先進的DRL方法**：我們與文獻 [31] 中提出的 DRL 方法進行了直接比較，並使用了其開源代碼進行訓練和測試。
    
   在測試階段，我們為 DRL 方法考慮了兩種動作選擇策略：一是「貪婪策略」，即總是選擇機率最高的動作；二是「採樣策略」，即對同一個實例平行採樣求解100次並記錄最佳結果，旨在「以可接受的計算負擔為代價來提高解的品質」。模型的效能主要透過兩個指標來評估：平均完工時間（makespan），以及其與已知最佳解之間的相對差距 。
    
   **C. 合成數據上的結果 (Results on Synthetic Data)**
    
   1. 訓練尺度內的效能 (Table I)
    
   在與訓練實例相同規模的測試集上，實驗結果非常明確。如表格 I 所示，「對於兩種數據集和所有問題規模，DANIEL 不僅顯著優於所有 PDRs，而且相較於 [31] 中的 DRL 解決方案，在兩種動作選擇策略下都表現出顯著的改進」。特別是在
    
   SD2 數據上，當加工時間範圍變大時，其他方法的效能受到影響，而「DANIEL 仍然表現良好，尤其是在使用採樣策略時」。最令人振奮的結果出現在
    
   SD1 的 20×10 實例上，我們的 DANIEL 演算法「甚至以 1.03% 的優勢擊敗了 OR-Tools」，而 OR-Tools 在時間限制內未能對該數據集中的任何實例找到最優解。從訓練曲線（圖 5）可以看出，相較於基準 DRL 方法，我們的模型「收斂到一個更好的解，且過程更為平滑，證實了其強大而穩定的效能」。
    
   2. 泛化至更大尺度的效能 (Table II)
    
   我們進一步檢驗了在小規模任務（
    
   10×5 和 20×10）上訓練出的模型的泛化能力，將它們直接應用於大規模的 30×10 和 40×10 測試實例 21。如表格 II 所示，「DANIEL 持續地為大規模問題生成高品質的解決方案，且計算成本可接受，這些解決方案顯著優於所有基準方法，並偶爾擊敗 OR-Tools」。一個極為亮眼的結果是，在
    
   SD1 的 40×10 實例上，由 20×10 實例訓練出的模型在使用採樣策略時，「比 OR-Tools 的表現好 6.60%」。這些結果表明，「DANIEL 可以透過在小規模任務上訓練來學習通用知識，這些知識可以被用來解決未見過的大規模實例」。
    
   **D. 公開基準測試集上的結果 (Results on Benchmarks)**
    
   對於一個模型而言，跨分佈應用能力至關重要 。因此，我們將在合成數據
    
   SD1 上訓練的模型應用於四組分佈完全不同的公開基準測試集 。
    
   如表格 III 所示，儘管 OR-Tools 和 2SGA 在效能上通常領先，但它們需要非常長的計算時間。相比之下，兩種 DRL 方法都取得了優於最佳 PDR 的良好效能，且運行時間在可接受範圍內。同樣地，「DANIEL 在大多數情況下都超過了 [31] 的方法，並在剩餘情況下表現出相當的效能」。特別是在 mk 實例上，DANIEL 在兩種選擇策略下都以較大優勢勝出。
    
   這些結果有力地證明了，「DANIEL 能夠真正捕捉到 FJSP 的內在結構資訊，並能辨別具有高優先級的相容配對，而不僅僅是學習特定數據分佈背後的規律」。我們相信，經過少量微調，DANIEL 在處理具有未知分佈的真實世界問題時能表現得更好 。
    

這一篇就大概是我的 baseline，找出這邊的資料然後做下去比較

<img src="{{ '/assets/notes/daniel-dual-attention/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/daniel-dual-attention/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/daniel-dual-attention/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />

## Related Notes

- Proximal Policy Optimization (PPO)
- Combinatorial Optimization

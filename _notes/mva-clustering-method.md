---
layout: page
title: "Clustering Method"
category: lecture
course: "Multivariate Analysis"
order: 3
---

### In general
- The goal of Clustering: 在沒有 label 的情況下找出資料中「自然的分組 (natural grouping)」。
- Clustering vs Classification:
	- **Clustering** (unsupervised)：不知道有幾群、也不知道哪個 item 屬於哪群。任務是 explore data structure。
	- **Classification** (supervised)：群數和成員都已知，目標是把新的 item 分到已知的某一群。
- 跟 [Principal Component Analysis]({{ '/notes/mva-principal-component-analysis/' | relative_url }}) 與 [Factor Analysis]({{ '/notes/mva-factor-analysis/' | relative_url }}) 的差別：PCA / FA 是在處理變數之間的線性關係 (variance / covariance)，clustering 則是在 **item 層級**找結構。要做 clustering 必須先有一個量化「相似度」的方式。
- 兩條路線 (這份 note 大多數方法都圍繞這個對立展開)：
	- **Cluster items**：用「距離」衡量 item 之間的接近程度。
	- **Cluster variables**：用「相關係數」衡量變數之間的關聯。

> **跟前兩個 topic 的銜接**
> PCA 把變數重排成 uncorrelated component，FA 把變數的 covariance 拆成共同因子 + 特殊因子。Clustering 不再對「變數空間」動手，而是把 $$n$$ 個 observation 看成 $$\mathbb{R}^p$$ 中的點集，去找這些點的群聚結構。三者其實是 multivariate analysis 的三種補位：PCA 重組 variance、FA 解釋 covariance、Clustering 找 grouping。

---
### Similarity Measures

###### Considerations
定義 similarity measure 時要考慮：
- 變數性質 (discrete / continuous / binary)
- 量測尺度 (nominal / ordinal / interval / ratio)
- subject matter knowledge

###### Distances for Pairs of Items
給定兩個 $$p$$ 維 item $$x = (x_1, \dots, x_p)^T$$、$$y = (y_1, \dots, y_p)^T$$，常見距離有：
- **Euclidean**:
$$ d(x, y) = \sqrt{(x_1 - y_1)^2 + \cdots + (x_p - y_p)^2} = \sqrt{(x - y)^T(x - y)} $$
- **Statistical distance** (帶 weighting matrix $$A$$，通常是 $$\Sigma^{-1}$$，等同 Mahalanobis):
$$ d(x, y) = \sqrt{(x - y)^T A (x - y)} $$
- **Minkowski metric**:
$$ d(x, y) = \left[\sum_{i=1}^p |x_i - y_i|^m\right]^{1/m} $$
	- $$m = 1$$: city-block (Manhattan) distance
	- $$m = 2$$: Euclidean

###### True distance 的條件
能用「真距離」最好：
1. $$d(x, y) = d(y, x)$$ (對稱性)
2. $$d(x, y) > 0$$ if $$x \neq y$$
3. $$d(x, y) = 0$$ if $$x = y$$
4. $$d(x, y) \leq d(x, z) + d(z, y)$$ (三角不等式)

但實務上多數 clustering algorithm 也吃 subjectively assigned distances。

---
###### Similarity Coefficients for Binary Items
- Binary 變數 1 / 0 通常代表「特徵存在 / 不存在」。直接用 squared Euclidean distance 等於數 mismatch 數，但這會把 1-1 match 和 0-0 match 視為同等重要，**很多時候不合理** (例如「兩人都沒有罕見疾病」不該跟「兩人都有」一樣有意義)。
- 為了給 1-1 / 0-0 / mismatch 不同權重，整理成 contingency table：

|  | $$j$$ = 1 | $$j$$ = 0 | Totals |
|---|---|---|---|
| $$i$$ = 1 | $$a$$ | $$b$$ | $$a+b$$ |
| $$i$$ = 0 | $$c$$ | $$d$$ | $$c+d$$ |
| Totals | $$a+c$$ | $$b+d$$ | $$p = a+b+c+d$$ |

- 8 種常用 coefficient：

| # | Coefficient | Rationale |
|---|---|---|
| 1 | $$\dfrac{a+d}{p}$$ | 1-1 與 0-0 等權重 |
| 2 | $$\dfrac{2(a+d)}{2(a+d)+b+c}$$ | 1-1 與 0-0 雙權重 |
| 3 | $$\dfrac{a+d}{a+d+2(b+c)}$$ | unmatched pair 雙權重 |
| 4 | $$\dfrac{a}{p}$$ | 0-0 不算 numerator |
| 5 | $$\dfrac{a}{a+b+c}$$ | 0-0 完全不算 (Jaccard) |
| 6 | $$\dfrac{2a}{2a+b+c}$$ | 0-0 不算，1-1 雙權重 (Dice) |
| 7 | $$\dfrac{a}{a+2(b+c)}$$ | 0-0 不算，mismatch 雙權重 |
| 8 | $$\dfrac{a}{b+c}$$ | match / mismatch ratio (excl. 0-0) |

- **Monotonic relations**:
	- Coefficients 1, 2, 3 互為 monotone 函數：若 $$\frac{a_I+d_I}{p} \geq \frac{a_{II}+d_{II} }{p}$$，則 coeff. 2、3 也滿足相同方向不等式。
	- 同理 5, 6, 7 也是 monotonically related。
- **意義**：在 hierarchical clustering 中只要 ranking 一樣就會給同樣的 dendrogram (對 single / complete linkage 而言)，所以這三組裡選哪一個其實效果一樣。

> **怎麼挑 coefficient**
> 沒有絕對答案。重點是想清楚：「兩個 item 都缺某個特徵」對你的問題有沒有資訊？有的話用包含 0-0 的 (1, 2, 3)；沒有 (例如 sparse 的 presence data) 就用 Jaccard 系列 (5, 6, 7)。

---
###### Similarity Measures for Pairs of Variables
- 對 **變數**做分群，常用 sample correlation 當 similarity，必要時把負相關取絕對值。
- 二元變數時，用同樣的 contingency table，sample correlation 是
$$ r = \frac{ad - bc}{[(a+b)(c+d)(a+c)(b+d)]^{1/2} }. $$
- 「item 用 distance、variable 用 correlation」是這份 note 的固定 convention。

---
### Hierarchical Clustering Methods
- 動機：找 reasonable cluster，但不需要枚舉所有 partition。
- 兩個方向：
	- **Agglomerative** (bottom-up)：每個 item 自己一群，每次合併最像的兩群，直到全部變成一群。
	- **Divisive** (top-down)：所有 item 一群，每次切出最不像的子群，直到每個 item 自己一群。
- 結果可以畫成 **dendrogram** (樹狀圖)，y 軸是合併 / 分裂時的距離。

#### Agglomerative Methods

###### Linkage Methods (cluster 與 cluster 之間的距離怎麼定？)
合併之後得到的新 cluster 跟其他 cluster 的距離，要用一個 linkage rule 算出來：
- **Single linkage** (nearest neighbor)：兩 cluster 中**最近**那對點的距離。
$$ d_{(UV)W} = \min\{d_{UW}, d_{VW}\} $$
- **Complete linkage** (farthest neighbor)：兩 cluster 中**最遠**那對點的距離。
$$ d_{(UV)W} = \max\{d_{UW}, d_{VW}\} $$
- **Average linkage**：兩 cluster 中**所有 pair 的平均**距離。
$$ d_{(UV)W} = \frac{\sum_{i \in (UV)} \sum_{k \in W} d_{ik} }{N_{(UV)} N_W} $$

###### Algorithm (for $$N$$ items)
1. 從 $$N$$ 個 single-item cluster 與 $$N \times N$$ 對稱距離矩陣 $$D = \{d_{ik}\}$$ 開始。
2. 在 $$D$$ 中找最近的一對 cluster $$U, V$$，距離為 $$d_{UV}$$。
3. 合併成 $$(UV)$$，更新 $$D$$:
	1. 刪掉 $$U, V$$ 的列與行。
	2. 加入 $$(UV)$$ 與所有剩下 cluster 的距離 (用所選的 linkage rule)。
4. 重複 step 2-3 共 $$N-1$$ 次，記錄合併順序與距離。

###### Properties of Different Linkages
| Linkage | 對距離 ranking 敏感嗎？ | 易產生什麼形狀？ |
|---|---|---|
| Single | **不敏感** (只要 ranking 不變，dendrogram 不變) | chain (拉長型) |
| Complete | **不敏感** (同上) | compact (緊密球狀) |
| Average | **敏感** (新距離可能改變排序) | 介於兩者之間 |

> **三種 linkage 的個性**
> Single 看「最強的橋」，所以兩群只要有一對點接近就會被連起來，容易形成 chain；complete 看「最弱的環」，所以一定要兩群整體都接近才會合併，傾向小而緊；average 折衷。實務上通常先三個都跑看 dendrogram 哪個最穩定。

---
###### Ward's Hierarchical Clustering
- 動機：每次合併都選「讓 within-cluster variance 增加最少」的一對 cluster。這個想法直接對應到 ANOVA / regression 中的 sum of squares 概念。
- 對 cluster $$k$$ 的 **error sum of squares (ESS)**：
$$ \mathrm{ESS}_k = \sum_{i=1}^{N_k} (x_i - \bar x_k)^T (x_i - \bar x_k), \qquad \bar x_k = \frac{1}{N_k} \sum_{i=1}^{N_k} x_i. $$
- 整體 $$\mathrm{ESS} = \mathrm{ESS}_1 + \cdots + \mathrm{ESS}_K$$。
- **Algorithm**:
	1. 初始：每個 cluster 只有一個 item，所有 $$\mathrm{ESS}_k = 0$$，總 $$\mathrm{ESS} = 0$$。
	2. 試遍所有 cluster pair 的合併，選讓 $$\mathrm{ESS}$$ **增加最少**的 pair 合併。
	3. 重複，直到剩一個 cluster。最後 $$\mathrm{ESS} = \sum_{i=1}^N (x_i - \bar x)^T(x_i - \bar x)$$ (整體變異)。

> **Ward's 跟 K-means 的隱性對應**
> Ward's 的 objective 跟 K-means 都是「最小化 within-cluster sum of squares」，差別在 Ward's 是 hierarchical (一旦合併就不能拆)，K-means 是 partitioning (每輪重新分配)。後面會看到，假設所有 cluster 共享 $$\Sigma_k = \eta I$$ 的 mixture model 也跟 K-means / Ward's 等價，三者其實在解同一個問題的不同變形。

---
#### Divisive Methods

###### Basic Division Algorithm (DIANA-like)
1. 在當前 cluster 裡找 **跟其他 item 平均不相似度最高**的那個 item，把它抽出來成立 splinter group $$A$$，剩下的是 $$B$$。
2. 對 $$B$$ 中每個 item 計算：
	- $$(1)$$: 跟 $$B$$ 中其他 item 的平均不相似度。
	- $$(2)$$: 跟 $$A$$ 中所有 item 的平均不相似度。
	- 算 $$(1) - (2)$$。
3. 若所有差都 $$\leq 0$$，停。否則把差最大且 $$> 0$$ 的 item 移到 $$A$$，回到 step 2。
4. 得到 binary split $$A, B$$ 之後，對 $$A, B$$ 各自再跑一次同樣流程，直到每個 cluster 只剩一個 item。

> **Divisive vs Agglomerative**
> Agglomerative 在 step 1 的 $$\binom{N}{2}$$ pair 中挑一對合併，計算量隨 $$N$$ 大致 $$O(N^2)$$ per step。Divisive 每次切會牽涉到 $$2^{N-1} - 1$$ 種可能 binary split，理論上更貴，所以實務上多用 agglomerative。

---
###### Comments on Hierarchical Methods
- **對 outlier 敏感**：因為沒有 model，沒有正式處理 error / variation。
- **不會 reallocate**：一旦合併或分裂就回不去了，前面的錯誤會一路帶下去。
- **建議**：同一份資料試多種方法 (single / complete / average / Ward's) 與多種距離，互相對照。
- **穩定性檢查**：在資料加上小擾動後重跑，看結果是否仍然一致。
- **Ties**：距離矩陣中相等的距離會產生多解，要意識到不同 tie-breaking rule 可能 dendrogram 完全不同。

###### Inversions
- **Inversion**: 後面合併的距離比前面合併的距離還小 (dendrogram 上出現「往下彎」的線)。
- 這會破壞 dendrogram 的 ultrametric 性質，讓圖形難以解讀。
- 主要發生在 **centroid method** 與 **median method** 上 (這兩個方法用 cluster 質心更新距離，新質心可能比原本兩個 cluster 都靠近某個 item)。
- 出現 inversion 通常表示資料根本沒有清晰的 cluster 結構。

---
### Partitioning Clustering Methods

#### K-means Method

###### Algorithm
1. 給定 $$K$$，要嘛 random partition $$N$$ 個 item 成 $$K$$ 群，要嘛指定 $$K$$ 個 seed point。
2. 對每個 item，把它指派給「**質心 (centroid) 最近**」的 cluster；質心隨之更新 (受影響的兩個 cluster 都要重算)。
3. 重複 step 2，直到沒有 reassignment。

###### 概念
- 等價於最小化 within-cluster sum of squares：
$$ \sum_{k=1}^K \sum_{i=1}^{N_k} (x_{ki} - \bar x_k)^T(x_{ki} - \bar x_k). $$
- K-means 找的是 **local minimum**，不是全域最佳；換 initial partition 重跑可能得到不同結果。
- 只能用於 **numeric** 資料 (因為要算 mean)。
- 對 outlier 敏感 (mean 會被拉走)。

> **K-means 跟 hierarchical 的本質差異**
> Hierarchical 一旦合併就不能拆，所以早期錯誤會永遠帶著。K-means 每輪都重新分配，理論上可以 escape 早期錯誤，但代價是 (1) 要先決定 $$K$$，(2) 結果隨 initial 變動，所以「跑很多次取最好的」幾乎是必要操作。

---
#### K-medoids Method (PAM)
- 動機：把 K-means 的「centroid」換成「medoid」(資料中真實存在的代表點)。
- **Medoid**: 一個 cluster 中跟其他成員平均不相似度最小的那個 item (i.e. 最中心的真實 data point)。
- 優點：
	- 對 outlier 比 K-means 更 robust (medoid 不像 mean 那樣會被拉走)。
	- 適用 categorical data (只要能定義 pairwise dissimilarity 就能跑，不需要算 mean)。
- **PAM (Partitioning Around Medoids) algorithm**:
	1. 從 $$N$$ 個 item 中選 $$K$$ 個當 medoid。
	2. 把每個非 medoid 指派給最近的 medoid。
	3. 對每個 cluster，看裡面哪個 item 換上去能讓 average dissimilarity 下降最多，就把它換成新 medoid (swapping)。
	4. 若有 medoid 改變，回 step 2；否則結束。
- 每輪 swapping 的 time complexity 是 $$O(K(n-K)^2)$$。

---
### Determining the Number of Clusters

挑 $$K$$ 跟 [Principal Component Analysis]({{ '/notes/mva-principal-component-analysis/' | relative_url }}) 中挑 PC 數類似：沒有絕對標準，要靠多種 heuristic 互相驗證。

#### Elbow Method
最小化 within-cluster variability 相對於 between-cluster variability。常見 criteria：
- **Wilks' lambda**: $$\dfrac{\vert B\vert }{\vert B + W\vert }$$ (越小越好)
- **Lawley-Hotelling trace**: $$\mathrm{tr}(BW^{-1})$$ (越大越好)
- **Within-cluster sum of squares**: $$\sum_{k=1}^K \sum_{i=1}^{N_k} (x_{ki} - \bar x_k)^T(x_{ki} - \bar x_k)$$ (越小越好)

把這些值對 $$K$$ 畫圖，找曲線出現「彎折 (elbow)」的位置 (跟 PCA 的 scree plot 同樣 spirit)。

#### Silhouette Method
評估每個 item 在當前分群下「歸對群了沒有」。對 item $$i$$ 定義
$$ s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\} } $$
其中：
- $$a(i) = \dfrac{1}{\vert C_I\vert  - 1} \displaystyle\sum_{j \in C_I, j \neq i} d(i, j)$$：$$i$$ 跟自己 cluster 內其他 item 的平均距離。
- $$b(i) = \displaystyle\min_{J \neq I} \dfrac{1}{\vert C_J\vert } \sum_{j \in C_J} d(i, j)$$：$$i$$ 跟「最近的別群」item 的平均距離。

性質：
- $$-1 \leq s(i) \leq 1$$。
- $$s(i) \approx 1$$：$$i$$ 很適合留在當前 cluster。
- $$s(i) \approx 0$$：在邊界上，模稜兩可。
- $$s(i) \approx -1$$：應該換到 neighbouring cluster。

###### Silhouette Coefficient (SC)
Kaufman 與 Rousseeuw 把 average silhouette 對 $$K$$ 做圖，**最大值**叫 silhouette coefficient，並給出主觀詮釋：

| SC | Interpretation |
|---|---|
| $$0.71 - 1.00$$ | strong structure |
| $$0.51 - 0.70$$ | reasonable structure |
| $$0.26 - 0.50$$ | weak / could be artificial |
| $$\leq 0.25$$ | no substantial structure |

> **Elbow vs Silhouette 怎麼選**
> Elbow 看「再加一群還能降多少 within-SS」，但「彎折點」常常很主觀 (尤其曲線平滑時)。Silhouette 直接量化「分群品質」，比較有絕對基準 (有 SC 表)。建議兩個都看、互相印證。

---
### Clustering Based on Statistical Model

###### Motivation
- Hierarchical 與 partitioning method 都是 intuitive 的 procedure，**沒有 model 解釋資料怎麼產生**。所以也無法討論「這個 cluster 是真的還是雜訊」。
- 引入 statistical model 之後就能用 likelihood 來做 inference (model selection、cluster assignment 都 model-based)。最常見的 model 是 **mixture model**。

###### Mixture Model
- $$X_i$$: 第 $$i$$ 個 object 的 $$p \times 1$$ 隨機向量。
- $$S_i \in \{1, \dots, K\}$$: 第 $$i$$ 個 object 的 latent cluster membership，$$P(S_i = k) = p_k$$。
- 邊際密度寫成 $$K$$ 個 component density 的加權平均：
$$ f_{\mathrm{Mix} }(x_i) = \sum_{k=1}^K P(S_i = k)\, f(x_i | S_i = k) = \sum_{k=1}^K p_k\, f_k(x_i) $$
其中 $$p_k \geq 0$$、$$\sum_{k=1}^K p_k = 1$$。
- 直觀：$$p_k$$ 是 cluster $$k$$ 的比例，$$f_k$$ 是 cluster $$k$$ 內資料的 density。資料就是「先抽 $$S_i$$，再從 $$f_{S_i}$$ 抽 $$x_i$$」這個雙層過程。

###### Normal Mixture Model
最常見的選擇：每個 component 都是 multivariate normal，$$f_k = N_p(\mu_k, \Sigma_k)$$。
$$ f_{\mathrm{Mix} }(x | \mu_1, \Sigma_1, \dots, \mu_K, \Sigma_K) = \sum_{k=1}^K p_k \frac{1}{(2\pi)^{p/2} |\Sigma_k|^{1/2} } \exp\!\left(-\tfrac{1}{2}(x - \mu_k)^T \Sigma_k^{-1}(x - \mu_k)\right) $$
- 每個 cluster 是 ellipsoidal，中心點密度最高 (這是 normal 的形狀，跟 Properties of multivariate normal 一致)。
- Likelihood (對 $$N$$ 個 object，$$K$$ 固定)：
$$ L(p_1, \dots, p_K, \mu_1, \Sigma_1, \dots, \mu_K, \Sigma_K) = \prod_{i=1}^N f_{\mathrm{Mix} }(x_i | \mu_1, \Sigma_1, \dots, \mu_K, \Sigma_K) $$
- 用 EM algorithm 解 MLE (因為 $$S_i$$ 是 latent，跟 [Factor Analysis]({{ '/notes/mva-factor-analysis/' | relative_url }}) 中 $$F$$ 是 latent 的處理 spirit 類似)。

###### 為什麼一定要對 $$\Sigma_k$$ 加限制？
完全自由的 $$\Sigma_k$$ 參數量為 $$K \cdot \frac{1}{2}(p+1)(p+2) - 1$$，當 $$N$$ 不夠大時會 over-parameterize。常見限制如下表：

| $$\Sigma_k$$ | 形狀 | 參數總數 |
|---|---|---|
| 無限制 | Ellipsoidal (各群獨立) | $$K \cdot \frac{1}{2}(p+1)(p+2) - 1$$ |
| $$\Sigma_k = \eta I$$ | Spherical, 等大小 | $$K(p+1)$$ |
| $$\Sigma_k = \eta_k I$$ | Spherical, 不同大小 | $$K(p+2) - 1$$ |
| $$\Sigma_k = \eta\,\mathrm{diag}(\lambda_1, \dots, \lambda_p)$$ | 軸對齊 ellipsoidal, 共享形狀 | $$K(p+2) + p - 1$$ |

更完整的分類 (mclust 中的 EII / VVV / VEV ... 系列) 用 $$\Sigma_k = \lambda_k D_k A_k D_k^T$$ 拆成 **volume / shape / orientation** 三個面向，每個面向各自決定要不要在 $$k$$ 之間共享。

> **跟 K-means / Ward's 的關係**
> 假設 $$\Sigma_k = \eta I$$ (所有 cluster 球狀且大小相等)，加上 equal mixing proportions，normal mixture model 的 MLE 跟 K-means / Ward's 的解幾乎一致。所以 K-means 可以視為 normal mixture model 的「最簡 case」。

---
###### Model Selection (AIC, BIC)
- 一旦把 clustering 嵌進 statistical model，「選 $$K$$」就變成「選 model」。標準工具就是 penalized likelihood：
$$ -2 \ln L_{\max} + \text{Penalty}. $$
- 對固定 $$K$$ 的 maximised likelihood:
$$ L_{\max} = L(\hat p_1, \dots, \hat p_K, \hat\mu_1, \hat\Sigma_1, \dots, \hat\mu_K, \hat\Sigma_K). $$
- **AIC**:
$$ \mathrm{AIC} = -2 \ln L_{\max} + 2 \cdot (\#\text{ of parameters}) $$
- **BIC**:
$$ \mathrm{BIC} = -2 \ln L_{\max} + 2 \ln(N) \cdot (\#\text{ of parameters}) $$
- 選 AIC (或 BIC) 最小的 model。BIC penalty 較重 (因為 $$\ln N \geq 2$$ 當 $$N \geq 8$$)，傾向選小一點的 $$K$$。

###### Cluster Labeling
有了 MLE 之後，對每個 object $$i$$ 算 posterior probability:
$$ \hat\theta_{ik} = \hat P(S_i = k | x_i) = \frac{\hat p_k\, \hat f_k(x_i)}{\sum_{k=1}^K \hat p_k\, \hat f_k(x_i)}. $$
把 $$i$$ 指派到 $$\hat\theta_{ik}$$ 最大的 cluster。這也是 EM 中 E-step 計算的 responsibility。

> **Soft vs hard assignment**
> $$\hat\theta_{ik}$$ 本身是「soft assignment」：它告訴你 object 在不同 cluster 之間的歸屬機率。跟 K-means 直接 hard assign 不同，mixture model 自然提供「邊界不確定的 object」這個概念，可以拿來做 outlier detection。

---
### Self-Organizing Maps (SOM)

###### What is SOM
- 由 Teuvo Kohonen 在 1980 年代提出，又稱 **Kohonen map**。
- 屬於 unsupervised neural network，但不是用 error backpropagation，而是 **competitive learning**。
- 靈感來自大腦皮質：相鄰神經元負責相似功能，輸出相似的神經元在空間上彼此靠近。
- 目標：把高維資料 project 到一個 **2D grid of nodes**，每個 node 代表一個小 cluster，且 grid 上相鄰的 node 對應原空間中相近的資料。

###### Output: SOM Plot
- 一張由 nodes (artificial neurons) 構成的網格圖，常用 hexagonal grid (視覺上比 rectangular 自然，相鄰關係對稱)。
- 每個 node $$v$$ 配一個跟 input 相同維度的 weight vector $$W_v$$；訓練後 $$W_v$$ 就是該 node 代表的 prototype。

###### Training Algorithm
每個 iteration $$t$$：
1. 抽一筆資料 $$X(t)$$。
2. 找 **best matching unit (BMU)**: 跟 $$X(t)$$ 最像的 node $$u$$ (通常用 Euclidean distance):
$$ u = \arg\min_v \|X(t) - W_v(t)\|. $$
3. 更新 BMU 與其 **鄰居** 的 weight vector:
$$ W_v(t+1) = W_v(t) + \eta(t)\,\phi(u, v, t)\,(X(t) - W_v(t)) $$
	- $$\eta(t)$$: learning rate (隨時間衰減)。
	- $$\phi(u, v, t)$$: neighborhood function，鄰居越遠值越小。
4. 重複直到收斂或達 max steps。

###### Common Choices
- **Learning rate**: $$\eta(t) = \eta_0 e^{-\lambda t}$$，初始 $$\eta_0$$、decay rate $$\lambda$$。
- **Neighborhood function** (Gaussian):
$$ \phi(u, v, t) = \exp\!\left(-\frac{\|u - v\|^2}{2\sigma(t)^2}\right), \qquad \sigma(t) = \sigma_0 e^{\beta t} $$
	- $$\sigma_0$$: 初始 neighborhood 寬度，$$\beta$$ 控制衰減 (注意原 lecture 寫 $$\sigma_0 e^{\beta t}$$，通常 $$\beta < 0$$ 才會衰減)。
- 最簡單的 neighborhood 也可以是 indicator: 鄰居 = 1, 其他 = 0，但 Gaussian 比較平滑。

###### Interpretation
SOM 訓練後常看的幾個圖：
- **Codes plot**: 每個 node 畫一張小圓餅 / 雷達圖，顯示該 node 的 weight vector 在各變數上的 profile，看 cluster 的特徵。
- **Counts plot**: 每個 node 上有幾筆資料被 map 過去；過熱的 node 可能 grid 太小，過冷的可能 grid 太大。
- **Quality plot**: 每個 node 對 mapped data 的平均距離 (越小越緊湊)。
- **Neighbour distance plot (U-matrix)**: 相鄰 node weight vector 的距離；高距離 = 兩塊 cluster 的「邊界」。

> **SOM 跟其他 clustering 的差別**
> K-means 給你 $$K$$ 個 hard label，SOM 給你 grid 上的 topology：除了「分到哪一群」，還告訴你「這群跟哪幾群相鄰」。所以 SOM 同時是 clustering + dimension reduction (類似 [Principal Component Analysis]({{ '/notes/mva-principal-component-analysis/' | relative_url }}) 投影到 2D)，但 SOM 是 non-linear 而 PCA 是 linear。

---
### Summary

###### 方法地圖
| 類別 | 代表方法 | 是否需要 model | 對 outlier |
|---|---|---|---|
| Hierarchical | Single / Complete / Average / Ward's | 否 | 敏感 |
| Partitioning (centroid) | K-means | 否 | 敏感 |
| Partitioning (medoid) | K-medoids (PAM) | 否 | 較 robust |
| Model-based | Normal mixture (EM) | **是** | 可調 (透過 robust component) |
| Topology-preserving | SOM | 否 (但有 architecture) | 中等 |

###### 工作流程建議
1. 先 EDA：看資料 scale、有沒有 outlier、變數性質 (continuous / binary / mixed)。
2. 選 similarity measure：item 用 distance、variable 用 correlation。
3. 多跑幾種 hierarchical method 比 dendrogram，初步感受結構。
4. 用 elbow / silhouette / BIC 三個角度估 $$K$$。
5. 用 K-means / K-medoids / mixture model 跑出 partition，多種 initial 取最佳。
6. 拿不同 method 的結果互相比對 (cross-tabulation)，看是不是 stable。
7. 最後 interpret cluster 時，務必對照 subject matter knowledge。

###### 跟 PCA / FA 的速查對照
| 面向 | PCA | FA | Clustering |
|---|---|---|---|
| 對象 | 變數空間 | 變數空間 | item 空間 |
| 性質 | linear transformation | statistical model | algorithm 或 model |
| 目標 | 重組 variance | 解釋 covariance | 找 grouping |
| 核心輸入 | $$\Sigma$$ (or $$R$$) | $$\Sigma$$ (or $$R$$) | distance / similarity matrix |
| Latent | 無 | $$F$$ (factors) | $$S$$ (cluster label, model-based 才有) |
| 估計 | eigen | PC method / MLE | linkage / K-means / EM |

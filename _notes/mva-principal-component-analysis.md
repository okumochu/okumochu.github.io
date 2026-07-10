---
layout: page
title: "Principal Component Analysis"
category: lecture
course: "Multivariate Analysis"
order: 1
---

### In general
- The goal of PCA: Dim reduction / Interpretation 
- 通常只需要共變異數矩陣，不用任何 Multivariate Normal (MVN) 的假設。因為其幾何意義就是轉軸，使其能夠變異最大化。
	- 但是如果是要做 inference，例如要檢測 sample PCA 的 eigen value 是否有顯著不同，就會需要 MVN 假設

### Population Principal Component

###### Definition
- 每一個 PC 都是 $$X = (X_1, \dots, X_p)^T$$  (共有 p 個 feature) 的 linear combination $$Y = a^TX$$ 
- 並且對於這個 $$Y$$ 有兩個限制下最大化變異 $$\mathrm{Var}(a_1^T X)$$
	- linear transformation is unit-norm $$a_i^T a_i = 1$$
		- 這個線性轉換在意的是要怎麼轉 variance 會最大跟長度沒有關係。在這個最大化問題框架下要無限放大 a，total variance 就會隨著無限放大。
	- 必須與前面所有 PC 不相關 $$\mathrm{Cov}(a_k^T X, a_i^T X) = 0, \; \forall k < i$$
		- 如果只要求 unit-norm 不要求正交的話，每個 PC 都會長一樣 (就是最大變異的軸 $$e_1$$) ，如果限制要找其他的方向就可以往其他軸去尋找

---
###### Derivation 
- Define the 1st PC as $$Y_1 = a_1^T X$$, where $$a_1 \in \mathbb{R}^p$$.
- Since $$\Sigma$$ is positive definite, by spectral decomposition its eigenvectors $$\{e_1, \dots, e_p\}$$ form an orthonormal basis of $$\mathbb{R}^p$$. Therefore any $$a_1$$ can be written as
	- 手法就是先將 $$a_1$$ 在某個 $$e_j$$ 的投影長度找出來 (轉換成 eigen basis)，接著再乘上那個方向 (又轉換成標準基底)
$$ a_1 = \sum_{j=1}^{p} \langle a_1, e_j \rangle\, e_j, \qquad \text{with } \|a_1\|=1. $$
- **Maximize $$\mathrm{Var}(Y_1)$$ subject to $$\Vert a_1\Vert =1$$:**
$$ \mathrm{Var}(Y_1) = a_1^T \Sigma a_1 = \sum_{j=1}^{p} \lambda_j \langle a_1, e_j \rangle^2, \quad \sum_{j=1}^{p} \langle a_1, e_j \rangle^2 = 1. $$

	1. $$\mathrm{Var}(Y_1) = \mathrm{Var}(a_1^T X) = a_1^T\, \mathrm{Cov}(X)\, a_1 = a_1^T \Sigma a_1.$$
	2. Let $$c_j = \langle a_1, e_j\rangle$$
$$ a_1 = \sum_{j=1}^{p} c_j\, e_j. $$
	3. Apply $$\Sigma$$ to $$a_1$$. Since $$\Sigma e_j = \lambda_j e_j$$, we have
		-  $$e_j$$ is one of the eigen vector of $$\Sigma$$, so it only scale by eigen value
$$ \Sigma a_1 = \sum_{j=1}^{p} c_j\, \Sigma e_j = \sum_{j=1}^{p} c_j \lambda_j\, e_j. $$
	4. Take the inner product with $$a_1^T$$, and use orthonormality ($$e_k^T e_j = 1$$ when $$k = j$$, otherwise $$0$$) to collapse the double sum into a single sum:
$$ a_1^T \Sigma a_1 = \Big(\sum_{k} c_k e_k\Big)^{\!T} \Big(\sum_{j} c_j \lambda_j e_j\Big) = \sum_{j,k} c_k c_j \lambda_j\, e_k^T e_j = \sum_{j=1}^{p} \lambda_j\, c_j^2 = \sum_{j=1}^{p} \lambda_j \langle a_1, e_j\rangle^2. $$
	5. The constraint $$\Vert a_1\Vert =1$$ collapses in the same manner:
$$ \|a_1\|^2 = a_1^T a_1 = \sum_{j,k} c_j c_k\, e_j^T e_k = \sum_{j=1}^{p} c_j^2 = \sum_{j=1}^{p} \langle a_1, e_j\rangle^2 = 1. $$

- Since $$\lambda_1 \geq \lambda_2 \geq \dots \geq \lambda_p$$, this is maximized by putting all the weight on $$e_1$$:
	- 上面的式子告訴你 total variance = eigen value weighted by $$c^2_j$$ and $\sum{c_j}^2 =1。所以為了在有限的權重下 maximize total variance，只好全部都給最大的那個 eigen value 
$$ a_1 = e_1 \;\Longrightarrow\; Y_1 = e_1^T X, \quad \mathrm{Var}(Y_1) = \lambda_1. $$
- 至於接下來的 PC 就會多一個 contraint 就是要跟 PC1 垂直，所以可想而知在這樣的條件下找到的 PC2 就是權重都給第二大的 eigen value:
$$ \mathrm{Cov}(Y_2, Y_1) = a_2^T \Sigma e_1 = \lambda_1\, a_2^T e_1 = 0 $$
-  $$\therefore a_2 \perp e_1$$. Maximizing $$\mathrm{Var}(Y_2) = a_2^T \Sigma a_2$$ in the subspace orthogonal to $$e_1$$ gives $$a_2 = e_2$$, so $$Y_2 = e_2^T X$$ with $$\mathrm{Var}(Y_2) = \lambda_2$$.
- By induction, the $$i$$-th PC is $$Y_i = e_i^T X$$.

> **證明思路**
> 這邊就是根據要 maximize 的東西慢慢寫下來，然後看看不同的 term 可不可以合併 (formulate $$a$$ with eigen vector) ，然後再把條件寫下來。最後觀察一下就可以得出結論了

---
###### Are PCs unique? (degree of freedom analysis)
Let $$X = (X_1, \dots, X_p)^T$$ with covariance matrix $$\Sigma$$.
- Number of free components in $$\Sigma$$ (symmetric): $$\dfrac{p(p+1)}{2}$$
	- 總共提供有上三角 ($$C^p_2$$) + 對角線 ($$p$$) 個方程式算出的純量
- Number of parameters introduced by PCA :
	- $$+p^2$$: 總共 p 個特徵向量，每個特徵向量有 p 個值
	- $$-p$$: 每個 unit norm 就是一條等式
	- $$-C^p_2$$: p 個 component 兩兩正交
$$ \underbrace{p^2}_{\#\, e_{ij} } + \underbrace{p}_{\#\, \lambda_i} - \underbrace{p}_{\|e_i\|=1} - \underbrace{\binom{p}{2} }_{\mathrm{Cov}(Y_i, Y_k)=0} = \frac{p(p+1)}{2} $$

**Answer**: Yes, the number of parameters equals the number of equations, so PCs are (essentially) uniquely determined.

> **證明思路**
>  要看 PC 是否 unique 就是看 $$e_i$$ 是否唯一 (indentifiable)，以代數的角度就是看未知數和方程式的個數，如果剛好兩個個數相等就是唯一。
>  PCA 基本上要解的參數個數就是 eigen value and eigen vector 的個數，然後需要扣掉一些限制。為了要解這個 PCA 提供的就是共變異數矩陣

---
###### Result

Let $$\Sigma$$ be the covariance matrix of $$X = [X_1, \dots, X_p]^T$$, with eigen-pairs
$$(\lambda_1, e_1), \dots, (\lambda_p, e_p)$$ satisfying $$\lambda_1 \geq \dots \geq \lambda_p \geq 0$$ and $$e_i^T e_i = 1$$.

Then the $$i$$-th principal component is
$$ Y_i = e_i^T X = e_{i1} X_1 + \dots + e_{ip} X_p, \quad i = 1, \dots, p, $$
with
$$ \mathrm{Var}(Y_i) = e_i^T \Sigma e_i = \lambda_i, \qquad \mathrm{Cov}(Y_i, Y_k) = e_i^T \Sigma e_k = 0, \; i \neq k. $$

---
###### Summary
$$ (X_1, \dots, X_p) \;\xrightarrow{\text{PCA} }\; (Y_1, \dots, Y_p) \quad \text{with} \begin{cases} \text{(1)} \;\; \mathrm{Cov}(Y_i, Y_j) = 0, \; i \neq j \\ \text{(2)} \;\; \mathrm{Var}(Y_i) = \lambda_i, \;\; \lambda_1 \geq \lambda_2 \geq \dots \geq \lambda_p \end{cases} $$

- 當 eigenvalue 真的很小時，可能只需要 $$(Y_1, \dots, Y_k)$$ 就能作為 $$(X_1, \dots, X_p)$$ 的 good approximation，這正是 PCA 做 dim reduction 的核心
- 另外，PCA 會被單位影響 (scale variate)，單位大的變異也會大
	- 如果想要對資料做 standardization 再做 PCA 等同於直接把 correlation matrix 拿來做 PCA 
	- 但是要注意，標準化過所以的變數都會變成同等重要，有時候說不定某些東西的變數就應該要比較重要 (單位是有意義的)
	$$\text{PCA on standardized } X \;\Longleftrightarrow\; \text{PCA on } \mathrm{Corr}(X)$$
- 最後 total variance 
	- = tr($$\Sigma$$) = tr($$P\Lambda P^{-1}$$) = tr($$PP^{-1}\Lambda$$) = tr($$\Lambda$$)
	- 如果是 Corr 則是 =  $$p$$ ，因為對角線上的值都是 1

---
###  Large Sample Inference
#unsolved

先去讀 FA 了，比較檢定的東西之後再慢慢補

---
### Factor Loading
- 每個 PC 就是原始變數的 linear combination：$$Y_i = e_i^T X = e_{i1} X_1 + \dots + e_{ip} X_p$$。因此 coefficient vector $$e_i = [e_{i1}, \dots, e_{ip}]^T$$ 的每個分量都值得被檢視，$$\vert e_{ik}\vert $$ 越大代表第 $$k$$ 個變數對第 $$i$$ 個 PC 的貢獻越大。
- 但是 $$e_{ik}$$ 本身只是「方向」上的權重，沒有考慮到 $$X_k$$ 本身 variance 多大、也沒有考慮到 $$Y_i$$ 的 variance（$$\lambda_i$$）有多大。要衡量 PC 和原始變數之間真正的**關聯強度**，更自然的是去看兩者的 correlation。

###### Correlation between $$Y_i$$ and $$X_k$$
- Result: 令 $$Y_1 = e_1^T X, \dots, Y_p = e_p^T X$$ 為由 $$\Sigma$$ 得到的 PCs，則 $$Y_i$$ 與 $$X_k$$ 的 correlation 為
$$ \rho_{Y_i, X_k} = \frac{e_{ik}\,\sqrt{\lambda_i} }{\sqrt{\sigma_{kk} } }, \qquad i, k = 1, \dots, p. $$
- 推導思路：
	1. $$\mathrm{Cov}(Y_i, X_k) = \mathrm{Cov}(e_i^T X, \; \mathbf{1}_k^T X) = e_i^T \Sigma \mathbf{1}_k = \lambda_i\, e_{ik}$$。其中用到 $$\Sigma e_i = \lambda_i e_i$$ 所以 $$e_i^T \Sigma = \lambda_i e_i^T$$。$$\mathbf{1}_k = (0, \ldots, 0, \underbrace{1}_{k}, 0, \ldots, 0)^T$$
	2. $$\mathrm{Var}(Y_i) = \lambda_i$$，$$\mathrm{Var}(X_k) = \sigma_{kk}$$
	3. 代入 correlation 定義 $$\rho_{Y_i, X_k} = \dfrac{\mathrm{Cov}(Y_i, X_k)}{\sqrt{\mathrm{Var}(Y_i)\cdot\mathrm{Var}(X_k)} } = \dfrac{\lambda_i e_{ik} }{\sqrt{\lambda_i \cdot \sigma_{kk} } }$$ 

###### Definition of Factor Loading
$$ \; \text{factor loading of } X_k \text{ on } Y_i \;=\; e_{ik}\sqrt{\lambda_i} \; $$
- 它就是 $$\rho_{Y_i, X_k}$$ 的分子，也就是**尚未用 $$X_k$$ 的標準差標準化**前的關聯量。
- 對於**標準化過**的變數 $$Z = (V^{1/2})^{-1}(X - \mu)$$（也就是對 correlation matrix 做 PCA），$$\sigma_{kk} = 1$$，所以
$$ \rho_{Y_i, Z_k} = e_{ik}\sqrt{\lambda_i} $$
	- 也就是在標準化情況下 **factor loading 就直接等於 correlation**，這也是為什麼實務上通常在標準化後才稱 $$e_{ik}\sqrt{\lambda_i}$$ 為 loading。

###### Intuition of Factor Loading
- **Loading 大 = 該變數和該 PC 高度相關**，所以可以拿來**解釋 PC 的意義**（e.g. 如果 $$Y_1$$ 在 `身高`, `體重`, `腿長` 的 loading 都很大且同號，就可以把 $$Y_1$$ 解釋為「體型」因子）。
- Loading 只衡量單一變數對 PC 的 **univariate contribution**，並沒有考慮其他變數的存在（not a partial correlation）。所以如果變數之間高度相關，loading 會同時給幾個變數很高的值，此時不能說它們都「獨立」地很重要。
- **Data reduction 的應用**：lecture 最後提到，PCA 不算真正「丟掉變數」，但可以透過**丟掉 loading 最小的原始變數**來減少變數數量，因為這些變數和主要 PCs 的關聯都很弱，資訊量相對少。

> 所以接下來介紹的 Factor Analysis 就是在做這件事情，找出不同變數群背後的一個潛在因子 (變數是這些潛在因子的線性表現) 來共同解釋變異

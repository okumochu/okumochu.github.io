---
layout: page
title: "Factor Analysis"
category: lecture
course: "Multivariate Analysis"
order: 2
---

### In general
- The goal of FA: 用一小撮 latent factors $$F_1, \dots, F_m$$ ($$m \ll p$$) 來解釋 $$p$$ 個觀測變數之間的 **covariance structure**。
- FA 的核心信念：「相關高的變數背後有共同的潛在因子」。把每個 $$X_i$$ 拆成「共同因子貢獻 (communality)」加「個別誤差 (specific variance)」，正是這個想法的數學化。
- FA vs PCA 的關鍵差異：
	- **PCA** 是純粹的 linear transformation：$$Y = E^T X$$，沒有 model assumption，目標是把總變異重新分配到正交軸上 (variance-maximizing rotation)。$$Y$$ 是 $$X$$ 的**直接函數**，永遠算得出來。
	- **FA** 是一個 **statistical model**：$$X - \mu = LF + \varepsilon$$，$$F$$ 是 latent (不可觀察)，目標是 fit **off-diagonal covariance**。Model 不一定 fit 得起來 (大多數 $$\Sigma$$ 其實沒辦法寫成 $$LL^T + \Psi$$)。
	- 一句話：PCA 重組 variance，FA 解釋 covariance。

> **接續 PCA 結尾的動機**
> 在 [Principal Component Analysis]({{ '/notes/mva-principal-component-analysis/' | relative_url }}) 的 factor loading 段落結尾提到：「Factor Analysis 在做的就是找出不同變數群背後的潛在因子」。這份 note 就是把那個 informal 的觀察 model 化。最自然的連結：PCA 中第一個 PC 上 loading 大的那群變數，正好對應 FA 中可能屬於同一個 common factor 的變數群。

---
### Orthogonal Factor Model

###### Definition
- 令 $$X = (X_1, \dots, X_p)^T$$ with $$E(X) = \mu$$, $$\mathrm{Cov}(X) = \Sigma$$。Orthogonal factor model 假設：
$$ \underbrace{X - \mu}_{p \times 1} = \underbrace{L}_{p \times m}\,\underbrace{F}_{m \times 1} + \underbrace{\varepsilon}_{p \times 1} $$
- 展開來逐個變數寫：
$$ \begin{cases} X_1 - \mu_1 = \ell_{11} F_1 + \ell_{12} F_2 + \cdots + \ell_{1m} F_m + \varepsilon_1 \\ X_2 - \mu_2 = \ell_{21} F_1 + \ell_{22} F_2 + \cdots + \ell_{2m} F_m + \varepsilon_2 \\ \quad\vdots \\ X_p - \mu_p = \ell_{p1} F_1 + \ell_{p2} F_2 + \cdots + \ell_{pm} F_m + \varepsilon_p \\ \end{cases} $$
- $$F = (F_1, \dots, F_m)^T$$：**common factors** (latent，所有變數共享)。
- $$\varepsilon = (\varepsilon_1, \dots, \varepsilon_p)^T$$：**specific factors / errors** (latent，每個變數獨有)。
- $$L = [\ell_{ij}]_{p \times m}$$：**factor loading matrix**，$$\ell_{ij}$$ 為第 $$i$$ 個變數在第 $$j$$ 個 factor 上的 loading。

###### Assumptions
1. $$F \perp \varepsilon$$ (independent)。
2. $$E(F) = 0$$，$$\mathrm{Cov}(F) = E(FF^T) = I_{m}$$。
	- factors 都是 mean 0、unit variance、彼此正交。「**orthogonal** factor model」名字就來自這條。
3. $$E(\varepsilon) = 0$$，$$\mathrm{Cov}(\varepsilon) = E(\varepsilon \varepsilon^T) = \Psi = \mathrm{diag}(\psi_1, \dots, \psi_p)$$。
	- 不同變數的 specific factors 之間互不相關。這是 FA 最核心的 model assumption：把「共有的部分」全部用 $$F$$ 解釋掉，剩下的 $$\varepsilon$$ 應該完全獨立。

> **為什麼可以假設 Cov(F) = I？**
> 看起來像很強的限制，其實是 normalisation。任何有相關 / 不同 variance 的 $$\tilde F$$ 都可以 whitening：用 $$\tilde F = A F$$ ($$A$$ 是某 invertible matrix) 變成 unit variance、互不相關，把多餘的 scale / rotation 吃進 $$L$$ 裡 ($$\tilde L = L A^{-1}$$)。所以「假設 $$\mathrm{Cov}(F) = I$$」沒有犧牲一般性，這也是後面 rotation indeterminacy 的根源。

---
###### Derivation of the Covariance Structure
- 目標：用 $$L, \Psi$$ 表達 $$\Sigma = \mathrm{Cov}(X)$$。
- Direct expansion:
$$ (X - \mu)(X - \mu)^T = (LF + \varepsilon)(LF + \varepsilon)^T = LFF^T L^T + \varepsilon F^T L^T + L F \varepsilon^T + \varepsilon \varepsilon^T. $$
- 取期望，利用 $$F \perp \varepsilon \Rightarrow E(F\varepsilon^T) = 0$$、$$E(FF^T) = I$$、$$E(\varepsilon \varepsilon^T) = \Psi$$：
$$ \Sigma = E[(X-\mu)(X-\mu)^T] = L \cdot I \cdot L^T + 0 + 0 + \Psi = LL^T + \Psi. $$
- 同樣方式可得 $$X$$ 與 $$F$$ 的 cross-covariance：
$$ \mathrm{Cov}(X, F) = E[(X-\mu)F^T] = E[(LF + \varepsilon)F^T] = L \cdot I + 0 = L. $$

###### Result (covariance decomposition)
$$ \boxed{\;\Sigma = LL^T + \Psi\;}, \qquad \boxed{\;\mathrm{Cov}(X, F) = L\;}. $$
- Element-wise:
$$ \sigma_{ii} = \mathrm{Var}(X_i) = \underbrace{\ell_{i1}^2 + \cdots + \ell_{im}^2}_{\text{communality } h_i^2} + \underbrace{\psi_i}_{\text{specific variance} } $$
$$ \sigma_{ik} = \mathrm{Cov}(X_i, X_k) = \ell_{i1}\ell_{k1} + \cdots + \ell_{im}\ell_{km} \quad (i \neq k) $$
$$ \mathrm{Cov}(X_i, F_j) = \ell_{ij}. $$
- **Communality** $$h_i^2 = \sum_{j=1}^m \ell_{ij}^2$$：$$X_i$$ 的 variance 中，**被 common factors 解釋掉**的那塊。
- **Specific variance / uniqueness** $$\psi_i$$：$$X_i$$ 獨有、無法被 common factors 解釋的那塊。
- $$\mathrm{Cov}(X_i, F_j) = \ell_{ij}$$：loading 同時也是「變數與因子之間的 covariance」，這給 loading 一個直觀的詮釋。
- **重點**：$$\sigma_{ik}$$ ($$i \neq k$$) 完全由 $$LL^T$$ 決定 (對角線以外 $$\Psi$$ 沒貢獻)。FA 真正想 fit 的就是 **off-diagonal**。

> **FA 的 model fit 角度**
> $$\Sigma$$ 共有 $$\dfrac{p(p+1)}{2}$$ 個獨立元素 (對稱)。對角的 $$p$$ 個 variance 一定可以用 $$h_i^2 + \psi_i$$ 湊出來 ($$\psi_i$$ 自由可調)，所以 FA 的 fit 好不好，重點在 **off-diagonal 的 $$\binom{p}{2}$$ 個 covariance** 是不是真的可以被低秩 $$LL^T$$ 表達。如果不能，FA model 就不適合這份資料。

---
###### Parameter Counting and Identifiability
- Number of free elements in target $$\Sigma$$: $$\dfrac{p(p+1)}{2}$$。
- Number of parameters in $$(L, \Psi)$$:
	- $$mp$$: factor loadings $$\ell_{ij}$$
	- $$p$$: specific variances $$\psi_i$$
	- 小計: $$p(m+1)$$
- 但 $$L$$ 不 unique，要再扣掉 rotation 的 $$\dfrac{m(m-1)}{2}$$ 自由度 ($$m \times m$$ orthogonal group $$O(m)$$ 的 dimension)。
- Effective degrees of freedom for FA to be identifiable:
$$ v_0 = \frac{p(p+1)}{2} - \left[p(m+1) - \frac{m(m-1)}{2}\right] = \frac{1}{2}\!\left[(p-m)^2 - (p+m)\right] $$
- 這個 $$v_0$$ 也正好是後面 LRT 的 degrees of freedom。需要 $$v_0 \geq 0$$ 才能進行檢定，等價於：
$$ m < \tfrac{1}{2}\bigl(2p + 1 - \sqrt{8p+1}\bigr). $$

> **直觀理解 identifiability**
> [Principal Component Analysis]({{ '/notes/mva-principal-component-analysis/' | relative_url }}) 中算過 PCA 的方程式跟未知數剛好相等，所以 PCs 唯一。FA 因為 $$m < p$$，方程式 ($$\Sigma$$ 的 entries) 多於未知數 ($$L, \Psi$$ 的 entries)，所以 model 通常 over-determined：**不是每個 $$\Sigma$$ 都能完美寫成 $$LL^T + \Psi$$**。這也是 lecture 強調「most covariance matrices cannot be factored as $$LL^T + \Psi$$ when $$m \ll p$$」的原因。

---
###### Rotation Indeterminacy (Why Rotation is Free)
- 令 $$T_{m \times m}$$ 為任意 orthogonal matrix ($$T T^T = T^T T = I_m$$)。Define
$$ L^* = LT, \qquad F^* = T^T F. $$
- 則 model 完全等價：
$$ X - \mu = LF + \varepsilon = (LT)(T^T F) + \varepsilon = L^* F^* + \varepsilon. $$
- 而且 $$F^*$$ 仍滿足 FA 的假設：
$$ E(F^*) = T^T E(F) = 0, \quad \mathrm{Cov}(F^*) = T^T \mathrm{Cov}(F) T = T^T I T = I. $$
- 對 covariance 的影響：
$$ \Sigma = LL^T + \Psi = (LT)(LT)^T + \Psi = L^* {L^*}^T + \Psi. $$
- **結論**：loadings 只能被決定到「差一個 orthogonal rotation」。
	- Communalities $$h_i^2 = (LL^T)_{ii} = (L^* {L^*}^T)_{ii}$$ **rotation-invariant** (對角元不變)。
	- Specific variances $$\psi_i$$ 也不變。
	- 但個別 loading $$\ell_{ij}$$ 會跟著 $$T$$ 一起變。
- 這個自由度就是後面 **factor rotation** 的合法性來源：既然所有 rotation 都同樣 fit data，那就挑一個「最好解釋」的版本。

---
### Methods of Estimation

主流兩種：**Principal Component method** 與 **Maximum Likelihood method**。Lecture 強調實務上會兩個都做、互相對照。

#### Principal Component Method

###### Derivation
- 對 $$\Sigma$$ 做 spectral decomposition (eigen-pairs $$(\lambda_i, e_i)$$，$$\lambda_1 \geq \dots \geq \lambda_p \geq 0$$)：
$$ \Sigma = \sum_{i=1}^{p} \lambda_i e_i e_i^T = [\sqrt{\lambda_1} e_1 \;\cdots\; \sqrt{\lambda_p} e_p] \begin{bmatrix} \sqrt{\lambda_1} e_1^T \\ \vdots \\ \sqrt{\lambda_p} e_p^T \end{bmatrix} = L L^T + 0. $$
- 也就是說：在 $$m = p$$、$$\Psi = 0$$ 的「退化情況」下，$$L$$ 的第 $$j$$ 行就是 $$\sqrt{\lambda_j} e_j$$，**正好是 PCA 中 $$X_k$$ 在 $$Y_j$$ 上的 factor loading** (見 [Principal Component Analysis]({{ '/notes/mva-principal-component-analysis/' | relative_url }}) 結尾)。
- 把後面 $$p - m$$ 個小 eigenvalue 丟掉做 approximation：
$$ \Sigma \approx \tilde L \tilde L^T + \tilde \Psi, \qquad \tilde L = [\sqrt{\lambda_1} e_1 \;\cdots\; \sqrt{\lambda_m} e_m], $$
$$ \tilde \Psi = \mathrm{diag}\!\left(\sigma_{11} - \sum_{j=1}^m \tilde\ell_{1j}^2, \;\ldots,\; \sigma_{pp} - \sum_{j=1}^m \tilde\ell_{pj}^2\right). $$
- $$\tilde \Psi$$ 用「對角線殘差」補回來，保證 $$\Sigma$$ 的對角線完全被 $$\tilde L \tilde L^T + \tilde \Psi$$ matched。但 **off-diagonal** 仍會有殘差，這就是 PC method 跟真正 FA 的妥協。

###### Sample Version
- 用 sample covariance $$S$$ (或 sample correlation $$R$$) 替換 $$\Sigma$$。eigen-pairs 寫成 $$(\hat\lambda_i, \hat e_i)$$：
$$ \hat L = [\sqrt{\hat\lambda_1}\, \hat e_1 \;\cdots\; \sqrt{\hat\lambda_m}\, \hat e_m], \qquad \hat \psi_i = s_{ii} - \sum_{j=1}^m \hat\ell_{ij}^2, \qquad \hat h_i^2 = \sum_{j=1}^m \hat\ell_{ij}^2. $$
- 兩個重要性質：
	1. **Loadings 不會隨 $$m$$ 改變**：把 $$m$$ 從 3 增加到 4，前 3 個 column 仍是同一組 $$\sqrt{\hat\lambda_j} \hat e_j$$，只是再加一行 $$\sqrt{\hat\lambda_4} \hat e_4$$。MLE 不具備這個性質。
	2. **Standardised case**：對 standardised $$Z$$ 做 PC method 等同於對 $$R$$ 做 PC method (跟 PCA 相同的 scale 議題)。

###### Selecting the Number of Factors $$m$$
$$ \text{Proportion of total sample variance due to } j\text{-th factor} = \begin{cases} \dfrac{\hat\lambda_j}{s_{11} + \cdots + s_{pp} } & \text{(based on } S\text{)} \\[4pt] \dfrac{\hat\lambda_j}{p} & \text{(based on } R\text{, since } \mathrm{tr}(R) = p) \end{cases} $$
- 三個常用 heuristic：
	1. 一直加 $$m$$ 直到累積比例「夠大」 (e.g. 80%)。
	2. **Kaiser's rule**: $$R$$ 的 eigenvalue 大於 1 的個數。直觀：standardised variance = 1，一個 factor 至少要解釋一個變數的份才值得留。
	3. **Scree plot**: 找 eigenvalue 曲線出現「彎折 (elbow)」的位置。

###### Residual Matrix
- Approximation 好不好可以看殘差矩陣 $$S - \hat L \hat L^T - \hat \Psi$$。
- Analytically, sum of squared entries 約等於被丟掉的 eigenvalues 平方和：
$$ \sum_{i,k}\!\left(S - \hat L \hat L^T - \hat \Psi\right)_{ik}^{\!2} \;\leq\; \hat\lambda_{m+1}^2 + \cdots + \hat\lambda_p^2. $$
- 所以「丟掉的 eigenvalues 都很小」 $$\Rightarrow$$ 「殘差矩陣的元素都很小」 $$\Rightarrow$$ approximation 良好。

---
#### A Modified Approach: Principal Factor Solution

###### Motivation
- 對 standardised case，若 model 正確，$$R$$ 的對角元素 $$1 = h_i^2 + \psi_i$$。
- PC method 直接對 $$R$$ 做 eigen 分解，等於把對角元素都當成 1 處理；但其實對角線「該被 common factors 解釋的部分只有 $$h_i^2$$」。
- **Principal factor solution** 的想法：先估個 $$\psi_i$$，把 $$R$$ 的對角線換成 $$h_i^2 = 1 - \psi_i$$，得到 reduced correlation matrix $$R_r$$，再對 $$R_r$$ 做 eigen 分解。

###### Procedure
$$ R_r = \begin{bmatrix} h_1^2 & r_{12} & \cdots & r_{1p} \\ r_{21} & h_2^2 & \cdots & r_{2p} \\ \vdots & \vdots & \ddots & \vdots \\ r_{p1} & r_{p2} & \cdots & h_p^2 \end{bmatrix} $$
- 對 $$R_r$$ 取前 $$m$$ 個 eigen-pairs $$(\hat\lambda_i^*, \hat e_i^*)$$，更新：
$$ \hat L = [\sqrt{\hat\lambda_1^*}\,\hat e_1^* \;\cdots\; \sqrt{\hat\lambda_m^*}\,\hat e_m^*], \qquad \hat\psi_i = 1 - \sum_{j=1}^m \hat\ell_{ij}^2, \qquad \hat h_i^2 = \sum_{j=1}^m \hat\ell_{ij}^2. $$
- 然後用新的 $$\hat\psi_i$$ 更新 $$R_r$$ 的對角線，**iteratively** 進行直到收斂。
- Initial estimate of $$\psi_i$$ 常用 $$1/r^{ii}$$，其中 $$r^{ii}$$ 是 $$R^{-1}$$ 的第 $$i$$ 個對角元 (對應 $$X_i$$ 對其餘變數做 multiple regression 後 $$1 - R_i^2$$ 的概念)。
- **注意**：因為 $$R_r$$ 對角線可能小於 1，eigenvalue 可能是負的，實務上要小心處理。

> **為什麼 principal factor 比 PC method 更貼近 FA**
> PC method 對 $$R$$ 直接做 eigen decomposition 是「用 PCA 模仿 FA」的一階近似，連對角線一起 fit。Principal factor 把對角線中「不該被 common factor 解釋」的 $$\psi_i$$ 先扣掉再做 eigen，這樣 eigen decomposition 就是真的在 fit off-diagonal covariance，spirit 上更接近 FA model 的本意。

---
#### Maximum Likelihood Method

###### Setup and Likelihood
- 進一步假設 $$F \sim N_m(0, I)$$、$$\varepsilon \sim N_p(0, \Psi)$$ 且 jointly normal，則
$$ X - \mu = LF + \varepsilon \;\sim\; N_p\!\bigl(0,\, LL^T + \Psi\bigr). $$
- 對 $$X_1, \dots, X_n$$ 寫 likelihood $$L(\mu, \Sigma)$$，再把 $$\Sigma$$ 限制成 $$LL^T + \Psi$$ 的形式之後 max over $$L, \Psi$$。
- 由於 $$L$$ 不 unique，ML 強制加一個 **uniqueness condition** 讓問題 well-posed：
$$ L^T \Psi^{-1} L = \Delta, \qquad \Delta \text{ diagonal}. $$
- MLE $$\hat L, \hat \Psi$$ 沒有 closed form，**只能 numerical optimisation**。
- By invariance of MLE：
$$ \hat h_i^2 = \hat\ell_{i1}^2 + \cdots + \hat\ell_{im}^2. $$
$$ \text{Proportion of total sample variance due to } j\text{-th factor} = \frac{\hat\ell_{1j}^2 + \cdots + \hat\ell_{pj}^2}{s_{11} + \cdots + s_{pp} }. $$

###### Standardised Version
- 若 $$Z = V^{-1/2}(X - \mu)$$，其中 $$V = \mathrm{diag}(\Sigma)$$，則
$$ \rho = V^{-1/2} \Sigma V^{-1/2} = \bigl(V^{-1/2} L\bigr)\bigl(V^{-1/2} L\bigr)^T + V^{-1/2} \Psi V^{-1/2}. $$
- 所以 standardised loadings 與 standardised specific variances 為
$$ \hat L_z = \hat V^{-1/2} \hat L, \qquad \hat \Psi_z = \hat V^{-1/2} \hat \Psi \hat V^{-1/2}. $$

---
###### Large Sample Test for the Number of Factors (LRT)
- Hypotheses (在 normality 下)：
$$ H_0: \Sigma = LL^T + \Psi \quad\text{with given } m \qquad\text{vs}\qquad H_1: \Sigma \text{ is any positive definite}. $$
- Maximised likelihoods:
	- 在 $$H_1$$ 下：$$\hat\mu = \bar x$$, $$\hat\Sigma = S_n = \dfrac{n-1}{n} S$$。Maximised likelihood $$\propto \vert S_n\vert ^{-n/2} e^{-np/2}$$。
	- 在 $$H_0$$ 下：$$\hat\mu = \bar x$$, $$\hat\Sigma = \hat L \hat L^T + \hat \Psi$$。Maximised likelihood $$\propto \vert \hat L \hat L^T + \hat\Psi\vert ^{-n/2} e^{-np/2}$$。
- Likelihood ratio statistic:
$$ -2 \ln \Lambda = n \ln \frac{|\hat L \hat L^T + \hat \Psi|}{|S_n|}. $$
- 大樣本下 $$\sim \chi^2_{v_0}$$，自由度
$$ v - v_0 = \tfrac{1}{2} p(p+1) - \left[p(m+1) - \tfrac{1}{2} m(m-1)\right] = \tfrac{1}{2}\!\left[(p-m)^2 - p - m\right]. $$
- **Bartlett's correction**：把 $$n$$ 替換成 $$n - 1 - (2p + 4m + 5)/6$$ 可改善 chi-square approximation：
$$ \bigl(n - 1 - \tfrac{2p + 4m + 5}{6}\bigr)\, \ln \frac{|\hat L \hat L^T + \hat \Psi|}{|S_n|} \;>\; \chi^2_{[(p-m)^2 - p - m]/2}(\alpha) \;\;\Rightarrow\;\; \text{reject } H_0. $$
- 因為自由度必須非負，可以 test 的 $$m$$ 上限是
$$ m < \tfrac{1}{2}\bigl(2p + 1 - \sqrt{8p+1}\bigr). $$

> **看 LRT 的角度**
> $$-2 \ln \Lambda$$ 越大 $$\Leftrightarrow$$ $$\vert \hat L \hat L^T + \hat \Psi\vert $$ 跟 $$\vert S_n\vert $$ 差越多 $$\Leftrightarrow$$ FA model 跟 sample covariance fit 得越差 $$\Leftrightarrow$$ 越要 reject「$$m$$ 個 factor 就夠用」這個假設。所以 reject = 因子數還不夠。

---
#### Comparison: PC method vs MLE
- **Residual matrix** $$R - \hat L \hat L^T - \hat \Psi$$ (lecture 例子)：
	- MLE 的 residual entries 都極小 (~$$10^{-3}$$ 量級)，因為 ML 直接把「fit 不上 covariance」這件事寫進 likelihood 然後最小化。
	- PC method 的 residual 明顯較大 (~$$10^{-1}$$ 量級)，因為它優先 fit 對角線 (variance) 而不是對角線外 (covariance)。
- **適用情境**：
	- PC method：簡單、不需 normality、loadings 不隨 $$m$$ 變，做 exploratory FA 很方便。
	- MLE：需要 normality，但可以做 LRT 來選 $$m$$，且 fit covariance 更精準。
- **Strategy** (lecture 結尾建議)：
	1. 先做 PC factor analysis。
	2. 再做 MLE factor analysis。
	3. 比較兩種結果。
	4. 對不同 $$m$$ 重複 1–3。
	5. 大資料時可以 split 然後分別跑。

---
### Factor Rotation

###### Motivation
- 由 rotation indeterminacy，給定估計 $$\hat L_{(p \times m)}$$，任何 $$\hat L^* = \hat L T$$ ($$T T^T = I$$) 都同樣 fit data。
- 那就挑一個 $$T$$ 讓 loading **解釋性最好**。理想的 pattern：「每個變數只在一個 factor 上 loading 大，其他都接近 0」，這樣每個 factor 就有明確的「主題」。
$$ \begin{aligned} X_1 - \mu_1 &= \underbrace{\ell^*_{11} }_\text{large} F_1 + \underbrace{\ell^*_{12} }_\text{small} F_2 + \cdots + \underbrace{\ell^*_{1m} }_\text{small} F_m + \varepsilon_1 \\ X_2 - \mu_2 &= \underbrace{\ell^*_{21} }_\text{small} F_1 + \underbrace{\ell^*_{22} }_\text{large} F_2 + \cdots + \underbrace{\ell^*_{2m} }_\text{small} F_m + \varepsilon_2 \\ &\;\;\vdots \end{aligned} $$
- Rotation **不會改變** communality $$\hat h_i^2$$、specific variance $$\hat \psi_i$$、$$\hat \Sigma$$，只把 loading 在 factors 之間「重分配」。

###### Graphical Method ($$m = 2$$)
- 當 $$m = 2$$，每個變數 $$i$$ 對應到平面上的點 $$(\hat\ell_{i1}, \hat\ell_{i2})$$。直接旋轉座標軸讓變數「貼齊」其中一軸：
$$ \hat L^* = \hat L \, T_{(2 \times 2)}, \qquad T = \begin{bmatrix} \cos\phi & -\sin\phi \\ \sin\phi & \cos\phi \end{bmatrix}\;(\text{counterclockwise rotation by } \phi). $$

###### Varimax Criterion (Kaiser, 1958)
- Step 1: scale loading by $$\hat h_i$$ (避免 communality 大的變數壓過其他):
$$ \tilde \ell_{ij} = \hat \ell_{ij} / \hat h_i. $$
- Step 2: 選 orthogonal $$T$$ 使下面的 $$V$$ 盡量大：
$$ V = \frac{1}{p} \sum_{j=1}^{m} \left[ \sum_{i=1}^p \tilde\ell_{ij}^4 - \frac{1}{p}\!\left(\sum_{i=1}^p \tilde\ell_{ij}^2\right)^{\!2} \right] $$
- 內層那一坨形式上就是「第 $$j$$ 個 factor 上各變數 (scaled) 平方 loading 的 variance」 $$\times p$$。
	- Variance 大 $$\Leftrightarrow$$ 各變數的平方 loading 落差大 $$\Leftrightarrow$$ 有些超大、有些很小 $$\Leftrightarrow$$ 達到「one large, others small」的目標。
- 把所有 factors 的 variance 加起來再最大化，就是 varimax 名字的由來 (variance maximisation)。

###### Oblique (Non-Orthogonal) Rotation
- 社會科學常用，不要求 rotated factors 之間互相正交 (允許 factors 有相關性)。
- 直觀對比：
	- **Orthogonal rotation** 是「rigid rotation」(剛性轉動，所有軸一起轉)。
	- **Oblique rotation** 是「non-rigid」(每個軸可以單獨往 cluster 方向斜過去)，因此能更貼近資料的 cluster。
- 代價：詮釋變得複雜 (factors 不再 independent)，後續 factor score、共變異重建公式都要改寫。

---
### Factor Score

###### What are factor scores?
- Estimated value of the unobserved $$F$$ for each observation $$j$$：
$$ \hat f_j = \text{estimate of the value } f_j \text{ attained by } F_j, \qquad j = 1, \dots, n. $$
- 注意 factor scores **不是參數的估計**，而是潛在隨機變數值的估計 (角色上類似 mixed model 中 random effect 的 BLUP)。

#### Weighted Least Squares (Bartlett) Method

###### Idea
- 把 $$\mu, L, \Psi$$ 都當成 known，$$F$$ 是要被估計的「未知值」。對單一觀測 $$x$$，model 給出
$$ x - \mu = L f + \varepsilon, \qquad \mathrm{Var}(\varepsilon_i) = \psi_i. $$
- 因為 $$\psi_i$$ 不一定相等 (heteroscedastic)，Bartlett 提議用 weighted LS：每個 component 用 $$1 / \psi_i$$ 做 weight。
$$ \sum_{i=1}^p \frac{\varepsilon_i^2}{\psi_i} = \varepsilon^T \Psi^{-1} \varepsilon = (x - \mu - Lf)^T \Psi^{-1} (x - \mu - Lf). $$
- 對 $$f$$ 求導等於 0，closed-form solution：
$$ \boxed{\;\hat f = (L^T \Psi^{-1} L)^{-1} L^T \Psi^{-1} (x - \mu).\;} $$

###### Plug-in Sample Version
- 把 $$\hat L, \hat \Psi, \hat \mu = \bar x$$ 帶回去，第 $$j$$ 個觀測的 factor score：
$$ \hat f_j = (\hat L^T \hat\Psi^{-1} \hat L)^{-1} \hat L^T \hat\Psi^{-1} (x_j - \bar x), \qquad j = 1, \dots, n. $$
- **MLE 特例**：MLE 滿足 uniqueness condition $$\hat L^T \hat\Psi^{-1} \hat L = \hat\Delta$$ (diagonal)，所以化簡為
$$ \hat f_j = \hat\Delta^{-1} \hat L^T \hat\Psi^{-1}(x_j - \bar x). $$
- **Standardised case** (factored $$R$$)：
$$ \hat f_j = (\hat L_z^T \hat\Psi_z^{-1} \hat L_z)^{-1} \hat L_z^T \hat\Psi_z^{-1}\, z_j, \qquad z_j = \hat D^{-1/2}(x_j - \bar x). $$
- **Rotation 之後**：用 $$\hat L^* = \hat L T$$ 計算的 factor scores 也只是被 $$T$$ 旋轉：$$\hat f_j^* = T^T \hat f_j$$。
- **PC method 特例**：因為 PC method 的「方向」跟 weighted LS 不太契合，習慣改用 unweighted LS，給出
$$ \hat f_j = (\tilde L^T \tilde L)^{-1} \tilde L^T (x_j - \bar x) = \begin{bmatrix} \hat e_1^T (x_j - \bar x) / \sqrt{\hat\lambda_1} \\ \vdots \\ \hat e_m^T (x_j - \bar x) / \sqrt{\hat\lambda_m} \end{bmatrix}. $$
也就是「前 $$m$$ 個 scaled principal components」 evaluated at $$x_j$$ (跟 PCA 的 score 直接對應)。

#### Regression Method

###### Idea
- 在 $$F, \varepsilon$$ jointly normal 的假設下：
$$ \begin{bmatrix} F \\ X - \mu \end{bmatrix} \sim N_{m+p}\!\left(0, \begin{bmatrix} I_{m \times m} & L^T \\ L & LL^T + \Psi \end{bmatrix}\right). $$
- 由 multivariate normal 的條件分佈公式 ($$F \vert  X$$ 也是 normal)：
$$ E(F | x) = L^T (LL^T + \Psi)^{-1}(x - \mu) = L^T \Sigma^{-1}(x - \mu), $$
$$ \mathrm{Cov}(F | x) = I - L^T \Sigma^{-1} L. $$
- 所以「regression-based factor score」就是 **conditional expectation** $$E(F\vert x)$$，相當於把 $$F$$ 對 $$X - \mu$$ 做 best linear predictor：
$$ \boxed{\;\hat f_j = \hat L^T (\hat L \hat L^T + \hat \Psi)^{-1}(x_j - \bar x).\;} $$
- 為了減少「$$m$$ 選錯」的影響，實務上常把 $$\hat L \hat L^T + \hat \Psi$$ 換成 sample covariance $$S$$：
$$ \hat f_j = \hat L^T S^{-1}(x_j - \bar x). $$

###### Standardised Case
$$ \hat f_j = \hat L_z^T R^{-1} z_j, \qquad j = 1, \dots, n. $$

#### Regression vs WLS
- 用 Woodbury-type identity:
$$ (LL^T + \Psi)^{-1} = \Psi^{-1} - \Psi^{-1} L (I + L^T \Psi^{-1} L)^{-1} L^T \Psi^{-1}. $$
- 兩種 score 之間的關係：
$$ \hat f_j^{LS} = \bigl(I + (\hat L^T \hat\Psi^{-1} \hat L)^{-1}\bigr) \hat f_j^R. $$
- **MLE 特例**：因為 $$\hat L^T \hat\Psi^{-1} \hat L = \hat\Delta$$ 是對角矩陣。當 $$\hat\Delta^{-1}$$ 的對角元都接近 0 時 (也就是 common factors「fit 得很強」時)，$$\hat f_j^{LS} \approx \hat f_j^R$$。
- 直觀對比：
	- **WLS**: frequentist angle，把 $$f$$ 視為 fixed unknown，最小化 weighted squared error。
	- **Regression**: Bayesian / shrinkage angle，把 $$f$$ 視為 random，求 conditional expectation，自動朝 0 shrink (因為 prior $$E(F) = 0$$)。

---
### Summary
- **FA model**:
$$ X - \mu = LF + \varepsilon \;\Longrightarrow\; \Sigma = LL^T + \Psi, \qquad \sigma_{ii} = \underbrace{h_i^2}_\text{communality} + \underbrace{\psi_i}_\text{specific var}. $$
- 重點觀念：**FA 在 fit covariance (off-diagonal)**，不像 PCA 那樣 fit variance (diagonal)。
- **PC method**：簡單、不需 normality、loading 隨 $$m$$ 不變。fit 對角線好但 off-diagonal 殘差大。
- **Principal factor solution**：在 $$R$$ 的對角線上把 $$\psi_i$$ 扣掉再做 eigen，spirit 上更接近 FA 本意。
- **MLE method**：需要 normality，可以做 LRT 選 $$m$$ (with Bartlett 修正)，fit 對 covariance 更精準。
- **Rotation**：因為 $$L$$ 只能定到 orthogonal rotation 為止，可以 freely rotate 到「容易解釋」的版本 (varimax 是最常用)。Rotation 不影響 communality / specific variance / $$\hat \Sigma$$。
- **Factor score**：兩種主流，Bartlett WLS 與 regression (conditional mean)。MLE 下兩者在 $$\hat\Delta^{-1}$$ 對角元接近 0 時近似相等。
- 實務 strategy：兩種 method 都跑、不同 $$m$$ 都試、必要時 split sample，互相對照才不會被單一方法騙。

> **跟 PCA 的對照速查**
> | 面向 | PCA | FA |
> |---|---|---|
> | 性質 | linear transformation | statistical model |
> | 目標 | 保留 variance | 解釋 covariance |
> | 假設 | 不需 (做 inference 才需 MVN) | 至少要 $$\Sigma = LL^T + \Psi$$；MLE 還需 normality |
> | $$Y$$ vs $$F$$ | $$Y = E^T X$$，可直接算 | $$F$$ latent，需估計 (factor score) |
> | Loading | $$e_{ik}\sqrt{\lambda_i}$$ (唯一) | $$\ell_{ij}$$ (差一個 rotation $$T$$) |
> | Rotation | 無意義 (PCs 已被 variance 排序固定) | 必要的解釋手段 |

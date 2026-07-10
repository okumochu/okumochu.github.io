---
layout: page
title: "Prerequisite"
category: lecture
course: "Multivariate Analysis"
order: 0
---

#linear_algebra #mathematical_statistics

### What is Eigen Decomposition

###### Eigen decomposition (對角化):  $$A = X \Lambda X^{-1}$$
- 其中 $$X = [e_1, \ldots, e_n]$$ 的各行是特徵向量，$$X$$ 未必是正交矩陣
- 並且 $$\Lambda$$ 是特徵值，是一個對角矩陣
- 幾何意義上來就是把一組向量做旋轉後 $$X^{-1}$$，拉伸$$\Lambda$$，再旋轉回去$$X$$

###### 如何判斷一個矩陣是否可以對角化
- 標準流程: 看對 $$A$$ 的每一個特徵值 $$\lambda$$，都有 $$\text{algebraic multiplicity} = \text{geometric multiplicity}$$
	- 對於每個 $$\lambda$$:
		- $$a(\lambda)$$ = $$\lambda$$ 在特徵多項式 $$\det(A - tI)$$ 裡的重根次數
			- 這個倍率 $$\lambda$$ **應該**佔多少個維度
		- $$g(\lambda) = \dim \ker(A - \lambda I) = n - \operatorname{rank}(A - \lambda I)$$（即 eigenspace 的維度）
			- 這個倍率**實際上**能提供的獨立純拉伸方向數量
		- 一般而言 $$1 \leq g(\lambda) \leq a(\lambda)$$ 永遠成立
		- 若 $$g < a$$ 稱為 defective，就 **不可** 對角化
	- 幾何上來說，對角化就是要把這個矩陣的行為，轉換成 $$n$$ 個彼此不相關的純拉伸
	- 若 $$g < a$$，代表這個 $$\lambda$$ 應該佔的維度沒有被填滿 → 整個空間**無法被純拉伸的方向鋪滿**
	- 缺的 $$a-g$$ 個方向會被 **shear** 補上，而 shear 不是對角矩陣能描述的動作，因為對角矩陣代表彼此不相關，但是 sheer 代表某個座標軸的值會彼此影響
	
- 實務上快速判斷的方法
	- 對稱 ($$A^T = A$$，實數)
		- 永遠可對角化，除此之外 $$X$$ 可選成 **Orthonormal Matrix** $$Q$$ ($$Q^T Q = I$$)，得 $$A = Q\Lambda Q^T$$，特徵值全為實數
	- Hermitian ($$A^* = A$$，複數)
		- 對稱的複數版本 ($$A^*$$ 是共軛轉置)
		- 永遠可對角化，$$X$$ 可選成 **Unitary Matrix** $$U$$ ($$U^* U = I$$)，特徵值仍全為實數
	- Normal ($$A^* A = A A^*$$)
		- 最一般的結構，涵蓋 unitary、orthogonal、Hermitian、skew-Hermitian。永遠可對角化，$$X$$ 可選成 unitary，但 **特徵值未必為實**
	- **結構強度**:  $$\text{Normal} \subset \text{Hermitian} \subset \text{Normal} \subset \text{可對角化}$$

- <mark>如果都不滿足，只能乖乖跑標準流程</mark>



---

### Given a real symmetric matrix $$A_{n\times n}$$

MVA - week 1, 第 4 頁
##### 為什麼 eigen value 必為實數
- 根據代數基本原理，在實數空間裡一元 n 次方程式存在 n 個解
- 代表特徵多項式 $$\det(A - \lambda I) = 0$$ 在 $$\mathbb{C}$$ 中恰有 $$n$$ 個根
- 因為 A 是實對稱矩陣，所以 $$A =\bar{A}= A^T$$ (取共軛就是虛部變號)
- 取共軛轉置 (conjugagte tanspose) $$A^* = \bar{A}^T = A$$
$$(Ax)^* = (\lambda x)^* \implies x^* A^* = \bar{\lambda}\, x^* \implies x^* A = \bar{\lambda}\, x^*$$
- 由原式 $$A x = \lambda x$$ **左乘** $$x^*$$
$$x^*(Ax) = x^*(\lambda x) \implies x^* A x = \lambda\, x^* x$$
- 由共軛式 $$x^* A = \bar{\lambda}\, x^*$$ **右乘** $$x$$
$$(x^* A)\, x = \bar{\lambda}\, x^* x \implies x^* A x = \bar{\lambda}\, x^* x$$
- 上下兩式相等並相減：
$$(\lambda - \bar{\lambda})\, x^* x = 0.$$
- 因 $$x \neq 0 \Rightarrow x^* x = \sum_i \vert x_i\vert ^2 > 0$$，故 $$\lambda = \bar{\lambda}$$，即 $$\lambda \in \mathbb{R}$$。$$\blacksquare$$


###### 為什麼能找到 $$n$$ set of eigen vector 且彼此正交
#unsolved 

**證明策略 (Rayleigh quotient / 變分法)**：

把「找特徵向量」轉化成「在單位球面上最大化 $$x^T A x$$」的**最佳化問題**。
- 每個 eigenvalue = quadratic form $$x^T A x$$ 在某個子空間上的極值
- 每個 eigenvector = 達到那個極值的方向
- 流程：在單位球面上找 $$x^T A x$$ 的最大點 → 得到第一個 eigenvector $$v_1$$；再把搜尋範圍限制到 $$v_1^\perp \cap$$ 單位球面，繼續找最大點 → 得到 $$v_2$$；依此做到 $$v_n$$

這個觀點直接連到 PCA：第一主成分 = 資料散度最大的方向 = 最大 eigenvalue 的 eigenvector。

---

**核心工具**

1. **Rayleigh quotient** $$R(x) = \dfrac{x^T A x}{x^T x}$$
	- 加分母是為了讓 $$R$$ 不受 $$x$$ 長度影響 (因為 $$x^T A x$$ 對 $$x$$ 的大小二次齊次，同方向的 $$x$$ 會被 scale)；等價於把 $$x$$ 限制在**單位球面** $$S^{n-1} = \{x : \Vert x\Vert =1\}$$ 上看 $$x^T A x$$
2. **Weierstrass 極值定理**：連續函數在緊緻集上必達到極大值與極小值
	- $$S^{n-1}$$ 在 $$\mathbb{R}^n$$ 中是有界閉集 → 緊緻
	- $$x^T A x$$ 是 $$x$$ 的多項式 → 連續
	- ∴ $$x^T A x$$ 在 $$S^{n-1}$$ 上**一定**達到極大、極小值 (**不需要依賴代數基本定理**，這是與代數派證法最大的分水嶺)
3. **Lagrange 乘子法**：在約束 $$g(x)=0$$ 下最佳化 $$f(x)$$，critical point 滿足 $$\nabla f = \sum_i \mu_i \nabla g_i$$
	- 會用到 $$\nabla_x (x^T A x) = 2 A x$$ (需要 $$A^T = A$$，否則梯度是 $$(A + A^T)x$$)

---

**Step 1. 第一個特徵向量 = 單位球面上 $$x^T A x$$ 的最大化點**

1. 定義目標 $$f(x) = x^T A x$$，約束 $$g(x) = x^T x - 1 = 0$$
2. 由 Weierstrass，$$f$$ 在 $$S^{n-1}$$ 上**必存在**最大值，記最大點為 $$v_1$$，最大值為
$$\lambda_1 := v_1^T A v_1 = \max_{\Vert x\Vert =1} x^T A x$$
3. 建構 Lagrangian:
$$\mathcal{L}(x, \mu) = x^T A x - \mu\,(x^T x - 1)$$
4. First-order condition ($$\nabla_x \mathcal{L} = 0$$):
$$2 A x - 2 \mu x = 0 \implies A x = \mu x$$
	- 這一步是整個證明的**魔法**：Lagrange 的梯度條件**直接**就是 eigenvalue 方程，無需額外構造
5. 所以最大點 $$v_1$$ 自動滿足 $$A v_1 = \mu v_1$$，即 $$v_1$$ 是 $$A$$ 的 eigenvector；對應的 Lagrange 乘子 $$\mu$$ 就是 eigenvalue
6. 用 $$v_1^T$$ 左乘兩邊確定 $$\mu$$ 的數值：
$$v_1^T A v_1 = \mu\, \underbrace{v_1^T v_1}_{=1} = \mu$$
	再由 $$\lambda_1$$ 的定義 $$v_1^T A v_1 = \lambda_1$$，得 $$\mu = \lambda_1$$

**得到**：$$(\lambda_1, v_1)$$ 是 eigenpair，而且 $$\lambda_1$$ 是所有 eigenvalue 中**最大**的 (因為 $$v_1$$ 是 global max)

---

**Step 2. 把搜尋範圍縮到 $$v_1^\perp$$，重做一次變分 → $$(\lambda_2, v_2)$$**

已經拿到 $$v_1$$，現在要第二個正交於 $$v_1$$ 的 eigenvector。自然的做法：把最佳化限制在 $$v_1$$ 的正交補上。

1. 新的可行集：$$S_2 = \{x \in \mathbb{R}^n : \Vert x\Vert =1,\ v_1^T x = 0\}$$
	- 這是 $$\mathbb{R}^n$$ 的 $$(n-1)$$ 維單位球面，仍是緊緻
2. 由 Weierstrass，$$x^T A x$$ 在 $$S_2$$ 上達到最大值 $$\lambda_2$$，最大點 $$v_2$$
3. Lagrangian 多了一個約束、多一個乘子：
$$\mathcal{L}(x, \mu, \alpha_1) = x^T A x - \mu\,(x^T x - 1) - \alpha_1\, v_1^T x$$
4. First-order condition:
$$2 A x - 2\mu x - \alpha_1 v_1 = 0 \quad(\star)$$
5. **關鍵：證明 $$\alpha_1 = 0$$** , 這樣 $$(\star)$$ 才會退化成 $$A x = \mu x$$
	用 $$v_1^T$$ 左乘 $$(\star)$$：
$$2\, v_1^T A x - 2\mu\, \underbrace{v_1^T x}_{=0\text{ (約束)} } - \alpha_1\, \underbrace{v_1^T v_1}_{=1} = 0$$
	其中第一項可由對稱性翻邊：
$$v_1^T A x \stackrel{A=A^T}{=} (A^T v_1)^T x = (A v_1)^T x = \lambda_1\, \underbrace{v_1^T x}_{=0} = 0$$
	- ==這步仰賴 $$A^T = A$$==：把 $$A$$ 從「$$x$$ 那邊」搬到「$$v_1$$ 那邊」吃掉 $$A v_1 = \lambda_1 v_1$$，再用 $$v_1 \perp x$$ 湊成零
	- 若 $$A$$ 非對稱，$$A^T v_1 \neq \lambda_1 v_1$$，這邊就歸不了零，Lagrange 乘子法直接崩掉
6. 所以 $$\alpha_1 = 0$$，$$(\star)$$ 變成 $$A v_2 = \mu v_2$$，而且 $$v_2 \perp v_1$$、$$\Vert v_2\Vert =1$$、$$\mu = \lambda_2$$

---

**Step 3. 一般化到第 $$k$$ 個 eigenvector**

同樣的邏輯重複：假設已得 $$v_1, \ldots, v_{k-1}$$ 兩兩正交、皆為單位 eigenvector，對應 $$\lambda_1 \geq \cdots \geq \lambda_{k-1}$$。

1. 可行集 $$S_k = \{x : \Vert x\Vert =1,\ x \perp v_1, \ldots, v_{k-1}\}$$ 是 $$(n-k)$$ 維單位球面，仍緊緻
2. $$\lambda_k := \max_{x \in S_k} x^T A x$$ 存在，記最大點為 $$v_k$$
3. Lagrangian:
$$\mathcal{L} = x^T A x - \mu(x^T x - 1) - \sum_{j=1}^{k-1} \alpha_j\, v_j^T x$$
4. First-order condition:
$$2 A x - 2\mu x - \sum_{j=1}^{k-1} \alpha_j v_j = 0 \quad(\star\star)$$
5. 對每個 $$j < k$$，用 $$v_j^T$$ 左乘 $$(\star\star)$$：
$$2\, \underbrace{v_j^T A x}_{= \lambda_j v_j^T x = 0} - 2\mu\, \underbrace{v_j^T x}_{=0} - \alpha_j\, \underbrace{v_j^T v_j}_{=1} - \sum_{i\neq j} \alpha_i \underbrace{v_j^T v_i}_{=0} = 0$$
	- 第一項用對稱性翻邊：$$v_j^T A x = (A v_j)^T x = \lambda_j\, v_j^T x = 0$$
	- 第二項由 $$x \in S_k$$ 的正交約束直接為 $$0$$
	- 第四項由歸納假設 $$v_j \perp v_i$$ 為 $$0$$
	- 剩下 $$\alpha_j = 0$$
6. 所有 $$\alpha_j$$ 全為 $$0$$，$$(\star\star)$$ 退化成 $$A v_k = \mu v_k$$，所以 $$v_k$$ 是 eigenvector，$$\mu = \lambda_k$$

---

**Step 4. 這個流程能走足 $$n$$ 次**

- 第 $$k$$ 次的可行集 $$S_k$$ 是 $$(n-k)$$ 維單位球面
- $$k=1$$: $$(n-1)$$ 維，有無窮多點；$$k=n-1$$: $$1$$ 維圓；$$k=n$$: $$0$$ 維球面，就是 $$\{\pm v_n\}$$ 兩個點 , 仍非空
- 所以可以連續取到 $$v_1, v_2, \ldots, v_n$$ 共 $$n$$ 個兩兩正交的單位特徵向量 ∎

---

**核心直覺**
- 變分觀點的精髓：**每個 eigenvalue = $$x^T A x$$ 在某層子空間上的極值**，每個 eigenvector = 實現那個極值的方向。這就是 PCA「第 $$k$$ 主成分 = 前 $$k-1$$ 主成分正交補上的最大散度方向」的直接來源
- 對稱性 $$A^T = A$$ 在證明裡只出現兩處、但都不可或缺：
	1. $$\nabla(x^T A x) = 2 A x$$ (讓 Lagrange 條件直接變成 $$Ax = \mu x$$)
	2. $$v_j^T A x = (A v_j)^T x = \lambda_j v_j^T x$$ (讓正交約束的乘子 $$\alpha_j$$ 全歸零)
- 不需要代數基本定理，改用**緊緻性 + 連續性**保證極值存在 , 這正是譜定理可以推廣到 infinite-dim Hilbert space 上 compact self-adjoint operator 的通道

**結論**：存在正交矩陣 $$Q = [v_1, \ldots, v_n]$$（$$Q^T Q = I$$）與對角矩陣 $$\Lambda = \operatorname{diag}(\lambda_1, \ldots, \lambda_n)$$ (且 $$\lambda_1 \geq \lambda_2 \geq \cdots \geq \lambda_n$$)，使得
$$A = Q \Lambda Q^T \qquad \blacksquare$$



---

### What is Definite

###### 只對 Symmetric (or Hermitian) 矩陣討論
- 只對 symmetric 討論的原因: 任何方陣 $$M$$ 都可以拆成以下兩個 term $$M = \underbrace{\tfrac{1}{2}(M+M^T)}_{S \text{ (symmetric)} } + \underbrace{\tfrac{1}{2}(M-M^T)}_{K \text{ (skew-symmetric)} }$$
- 其中 skew-symmetric 部分對 $$x^T M x$$ 沒有貢獻，所以 quadratic form 的資訊**完全**藏在對稱部分
	- skew-symmetric: $$K^T = -K$$ (且對角線沒有值)
	- quadratic from is a scaler 所以 scaler tranpose 後不變
	- $$x^T K x = (x^T K x)^T = x^T K^T x = -x^T K x \implies x^T K x = 0$$
	- 又因為自己等自己的加負號，所以就只能 = 0
- 對稱矩陣 $$A$$ 配上一個向量 $$x$$，可以定義 **quadratic form (二次型)**
$$Q(x) = x^T A x = \sum_{i,j} a_{ij}\, x_i x_j$$
###### 各種 definite 的分類

| 名稱                                                | 條件                                  | eigenvalue            |
| ------------------------------------------------- | ----------------------------------- | --------------------- |
| Positive Definite                                 | $$x^T A x > 0$$ $$,\forall$$ $$x \neq 0$$ | 全部 $$\lambda_i > 0$$    |
| Positive Semi-Definite<br>(non-negative definite) | $$x^T A x \geq 0$$ $$,\forall$$ $$x$$     | 全部 $$\lambda_i \geq 0$$ |
| Negative Definite                                 | $$x^T A x < 0$$ $$,\forall$$ $$x \neq 0$$ | 全部 $$\lambda_i < 0$$    |
| Negative Semi-Definite (NSD)                      | $$x^T A x \leq 0$$ $$,\forall$$ $$x$$     | 全部 $$\lambda_i \leq 0$$ |
| Indefinite                                        | 既能為正也能為負                            | 有正有負                  |

##### 為什麼決定 definite 條件和 eigenvalue 等價
- 因為是對稱矩陣所以可以對角化 $$A = Q\Lambda Q^T$$，令 $$y = Q^T x$$ (換坐標系，因為 Q 是 orthonormal matrix)，則
$$x^T A x = x^T (Q\Lambda Q) x= (x^TQ) \Lambda (Q^Tx)= y^T \Lambda y = \sum_i \lambda_i y_i^2$$
  這個式子的正負完全由 $$\lambda_i$$ 的正負決定，也代表如果 $$A$$ 是 p.d. $$\implies$$ quadratic form > 0 ，所有的 eigen value > 0

###### 幾何意義：看 level set $$\{x : x^T A x = c\}$$
<img src="{{ '/assets/notes/mva-prerequisite/Pasted-image-20260418180910.png' | relative_url }}" alt="Pasted image 20260418180910" loading="lazy" style="max-width: 100%; height: auto;" />

| Type           | Optimization meaning                        | Origin is              | Shape         |
| -------------- | ------------------------------------------- | ---------------------- | ------------- |
| Positive def.  | strictly convex, unique minimizer           | strict min             | bowl          |
| Negative def.  | strictly concave, unique maximizer          | strict max             | inverted bowl |
| Singular & PSD | convex but not strict, minimizer not unique | min along a whole line | valley        |
| Indefinite     | neither convex nor concave                  | saddle point           | saddle        |

MVA - week 1, 第 5 頁
若一個矩陣 = p.d. 
- $$\iff$$ 代表該矩陣為對稱且 quadratic form > 0
- $$\implies$$ 該矩陣之 eigen value > 0

### Matrix Operation

> $$\text{Cov}(AX) = A\,\text{Cov}(X)\,A^\top$$ if $$A$$ is a non-random (deterministic) matrix
> MVA - week 1, 第 10 頁
> 
>  Given constant matrices A and B, we have $$Cov(AX , BY ) = A Cov(X , Y )B^⊤$$
>  MVA - week 1, 第 12 頁

1. Let $$\mu_X = E[X]$$, $$\mu_Y = E[Y]$$, then $$E[AX] = A\mu_X$$ and $$E[BY] = B\mu_Y$$
2. $$\text{Cov}(AX, BY) = E[(AX - A\mu_X)(BY - B\mu_Y)^\top] = E[A(X-\mu_X)(Y-\mu_Y)^\top B^\top]$$
3. $$= A\,E[(X-\mu_X)(Y-\mu_Y)^\top]\,B^\top = A\,\text{Cov}(X,Y)\,B^\top$$


> Corr(X ) = diag( 1 √σ11 , . . . , 1 √σpp ) × Cov(X ) × diag( 1 √σ11 , . . . , 1 √σpp )
MVA - week 1, 第 11 頁


剩下還沒看完先去看 PCA and FA
#unsolved

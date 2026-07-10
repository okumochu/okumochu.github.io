---
layout: page
title: "Monte Carlo Tree Search: A Review of Recent Modifications and Applications"
description: "Survey walkthrough of the MCTS phases and variants, with my notes on relevance to scheduling."
category: research paper
order: 1
---

****

## Introduction
> Monte Carlo Tree Search (MCTS) is a decision-making algorithm that consists in searching combinatorial spaces represented by trees.

- AlphaGO 在應用 MCTS 的時候都會把強化學習當作是 backbone。並且不只是圍棋遊戲，MCTS 幾乎應用在所有的 combinatorial game 裡面 e.g., poker, chess, ...
- 所以當然除了遊戲之後 plannig, scheduling, combinatorial optimization 經常會使用，更精確的是問題可以被描述成 MDP 就可以應用到 MCTS
	- MDP: Discrete-time stochastic control process

---

## Methodology

最原始的 MCTS 是透過  randomn rollout  的統計量的 tree policy 來 branch out, selection
1. Rollout 到底 (monte carlo) 這件事情非常耗費資源
2. 隨機 rollout 在困難的問題 (每步上千種分支) 上探索效率非常低
3. 不能解 continuous environment，但是 RL policy 可以
所以這個 tree policy希望能用 NN 來訓練

### Classic MCTS (Section 2)

Each MCTS iteration has **four phases**:
1. **Selection:** Traverse existing tree from root using **tree policy** to select a leaf node
2. **Expansion:** Add one or more child nodes to the tree
3. **Simulation**: Random playout from the new node to a terminal state (the "Monte Carlo" part)
4. **Backpropagation**: Propagate the "score" back along the path from leaf to root, updating statistics
> MCTS 叫做 "anytime" algorithm 這個方法能在任何時候停止給你目前最好的答案

<img src="{{ '/assets/notes/mcts-review/Pasted-image-20260411152200.png' | relative_url }}" alt="Pasted image 20260411152200" loading="lazy" style="max-width: 100%; height: auto;" />
<img src="{{ '/assets/notes/mcts-review/Pasted-image-20260411153014.png' | relative_url }}" alt="Pasted image 20260411153014" loading="lazy" style="max-width: 100%; height: auto;" />


### Tree Policy
目的在於設計出統計量使其在 selection 的過程能夠平衡 exploration and exploitation
最常使用的就是 **UCT (Upper Confidence bounds applied to Trees)**，也就是 multi-armed bandit 問題中的 UCB1 演算法，Kocsis & Szepesvári (2006) 把它搬到 tree search 上使用。其特性為隨著模擬次數趨近無限，UCT 會收斂到 optimal policy

這是 UCT 包含以下幾個 term
1. 多少次經過這個 state 以及
2. 通過這個 state select 這個 action 的次數
3.  $$Q(s,a)$$ 跟強化學習的一模一樣: 預期未來會得到的 reward 
4. Constant C 越大就是越能探索，因為代表拜訪次數少的有加成

$$a^* = \arg\max_{a \in A(s)} \left\{ Q(s,a) + C\sqrt{\frac{\ln[N(s)]}{N(s,a)} } \right\}$$

但是後面就有發展出各式各樣的 policy 依據需求，沒有最好只有最適合。多數的修改 (add bias) 只在該問題會變好，去其他問題反而變糟

| Mechanism                | Description                                                                                                                                  |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **UCT**                  | Balances exploration vs. exploitation using the UCB1 formula                                                                                 |
| **UCB1-Tuned**           | Improves UCT by also considering the variance of rewards: caps the exploration bonus so uncertain actions don't get over-explored            |
| **EXP3**                 | Makes no assumption on reward distribution; uses exponential weighting to handle partial observability (adversarial settings)                |
| **Thompson Sampling**    | Bayesian approach: maintains a belief (posterior) over reward, samples from it to decide , naturally balances explore/exploit                |
| **RAVE**                 | Shares action stats across sibling subtrees for faster early estimates; blends RAVE value with real value, trusting RAVE less as visits grow |
| **Transposition Tables** | Detects duplicate states and merges them into a DAG, avoiding redundant search of the same position reached via different move orders        |
| **History Heuristic**    | Tracks global success stats per action across the whole tree; biases rollout policy toward historically good moves (ε-greedy or Boltzmann)   |

## Games with Perfect Information

這個部分在說明如果是 Perfect Information (可以看到所有 state，沒有東西是被藏起來的。像是圍棋，反例就是 Poker) 的狀況要如何 improve MCTS。這些遊戲的問題在於 search space 超級大

### Action Reduction

這個方法主要是處理在每一步可以採取的 action 實在太多使用的，因為這樣會讓樹太寬，有限的計算資源下無法深層探索。
其他的方法大致上就是 
1. action abstraction: 把多部 action 包在一起，中間就不會分支
2. action reduction: 不要 branch out 所有分支

| Method                     | Key Idea                                                                        |
| -------------------------- | ------------------------------------------------------------------------------- |
| **Heuristic Move Pruning** | Use domain heuristics (card stats, heat maps) to prune actions                  |
| **Script-based Reduction** | Assign action scripts to unit groups instead of individual unit actions         |
| **Option MCTS**            | Replace atomic actions with predefined "options"                                |
| **PGSS**                   | Constraints (prune losing actions) + Options (bias toward desired trajectories) |
| **Informed MCTS**          | Use Calibrated Naive Bayes / Action-Type Independence models as priors          |
| **Beam MCTS**              | Keep only top-$$W$$ most promising nodes at depth $$d$$, prune the rest             |
| **VSP-MCTS**               | Variable simulation periods; progressively widen action set                     |
| **Progressive Bias**       | Extend UCB with heuristic value that decays with visit count                    |

**Macro-actions (skill)** predefined, not learned
<img src="{{ '/assets/notes/mcts-review/Pasted-image-20260411172323.png' | relative_url }}" alt="Pasted image 20260411172323" loading="lazy" style="max-width: 100%; height: auto;" />

### UCT Alternatives
要解決 UCT 在 perfect information 遊戲中的問題
1. **Cold-Start Problem:** 初期 UCT 都是在隨機探索，為了能夠加快收斂，論文提到很多時候一個 action 的好壞不需要 depend on state。在這樣的基礎下就可以共享統計量，並在後期慢慢收斂的時候，再把真實的更新權重還回真實的 MC 估計
2. **Optimistic Moves:** UCT 發現了一條看起來能贏的路徑，就不斷回去探索它、給它高分。但對手其實只需要一步就能破解。在 MCTS 找到那個反駁之前，它會持續被這條假路徑誤導。
3. **Average loophole:** 一個 state 下有 100 次 simulation，其中 95 次贏、5 次輸。UCT 會給它很高的平均分。但如果那 5 次輸的路徑對手<mark>是一定會走的最佳反擊</mark>，那這個 state 其實是必輸的。平均值完全掩蓋了這個事實，所以解法就是 Minimax
4. **Non-uniformity of tree shape:** 當搜索樹的形狀不均勻 (某些分支深、某些淺)，再計算分數上會有問題，因為不同深度的分數 (+10 分) 變異是不同的，越多步數變異越大。所以步數大的會有這個問題
	- 解法就是把所有分數畫出來並且找一個 $$\theta$$  (能最平均分配) 當作分水嶺來二元區分贏還輸，這個就是 <mark>optimal threshold selection</mark> ，
	- 優點: robustness to outlier, scale invariance, bounded variance (Bernoulli)
	Note: 這種會在遊戲中比較常出現，因為做錯選擇導致很快就輸掉。但是組合最佳化問題不太會有這樣的狀況發生因為該有多少個實體要排序就是固定的

| Method                    | Modification                                                                                             |
| ------------------------- | -------------------------------------------------------------------------------------------------------- |
| **AMAF / RAVE**           | Share action values across subtrees using all-moves-as-first heuristic                                   |
| **Sufficiency Threshold** | Set $$C=0$$ when any $$Q(s,a) > \tau$$ to reduce exploration of "solved" states                              |
| **MCTS-Minimax Hybrids**  | Shallow minimax search within MCTS phases to catch tactical traps                                        |
| **Maximum Frequency**     | Adjust threshold for recalculating playout scores to maximize difference between optimal and other moves |

### Early Termination

Terminate simulations at a **cut-off depth** instead of playing to terminal state, using an **evaluation function** $$v(s)$$
Even a **weak evaluation function** can outperform long random playouts (Lorentz 2016). Neural networks can approximate evaluation functions, allowing omission of the simulation step entirely (Goodman 2019).
下面這個就是其中一個設計出來的例子
$$Q'(s,a) = (1-\alpha) \frac{r_{s,a} }{n_{s,a} } + \alpha \cdot v_{s,a}$$
### Games with Imperfect Information (Section 4)

這個部分就是在探討 Imperfect information condition，此時不像是 PI 問題在於爆炸性成章的 search space，而是 patially observation。因為資訊不完整的狀況下連樹長什麼樣子都不確定。其中有幾個解法 <mark>以下我是用AI 摘要</mark>
1. **Determinization (PIMC):** 隨機假設 hidden state e.g., 假設對手的牌是什麼
	- 大量重複試驗來統計結果，但是會造成的問題是
		- Strategy Fusion: 平均下來說策略可能會變。假設 50% 機率對手有 A 牌（最佳應對：打 X）、50% 有 B 牌（最佳應對：打 Y）。平均後可能選了 Z，但 Z 在兩種情況下都不好
		- Non-locality: 每次 determinization 是獨立求解的，無法發現「試探性」的 action（為了蒐集資訊而做的次優 action）。例如在撲克中刻意加注來測試對手牌力，PIMC 永遠不會發現這種策略
		- Budget Splitting: 計算資源被分散到多棵獨立的搜索樹中，每棵樹分到的模擬次數變少，搜索品質下降
2. **Information Sets (ISMCTS):** 既然 PIMC 的問題在硬把 imperfect info 轉成 perfect info，不如直接在 **information set** 上建樹搜索。一個 information set 就是從當前玩家角度所有不可區分的 state 的集合
	- 每次迭代開始時從 information set 中抽樣一個 determinization 來做 selection/expansion/simulation，但 backpropagation 在**同一棵樹**上進行，解決了 budget splitting
	- 變體: **OOS** (Online Outcome Sampling) 透過 counterfactual regret minimization 能收斂到 **Nash Equilibrium**，理論保證不可被剝削；**IIMCTS** 用遞迴方式防止 simulation 階段的 information leakage
3. **Heavy Playouts:** 隨機 playout 在 imperfect info 遊戲中品質特別差，因此用 domain knowledge 來引導 simulation 使其更接近真實遊戲
	- 關鍵發現: <mark>完全確定性的 expert playout 反而比帶有隨機性的簡化規則 playout 表現更差</mark>，因為失去了 MC 方法靠多樣性來探索的優勢。最佳做法是在 domain knowledge 和 randomness 之間取得平衡
4. **Policy Update:** 在搜索過程中動態更新 playout policy，使模擬品質隨時間提升
	- **RIDE**: 比較 action pair 的相對優勢 $$D_{\text{RIDE} }(a,b) = E\{Q(s,a) - Q(s,b)\}$$，比絕對值更穩定
	- **Weighted Playouts**: 權重按指數函數更新 $$P(a) \propto e^{w(a)}$$，好的 action 被指數級放大
	- **Value Networks for Cutoff**: 用 NN 預測勝率來截斷 simulation，大幅減少長度把預算省給更多 iteration
5. **Master Combinations:** 最強的系統都是多種增強方法的組合。Vanilla MCTS 在 GVGP 只有 **31%** win-rate，組合 macro-actions + policy updates + knowledge base 後提升到 **48.4%** (Soemers et al. 2016)

## MCTS + Machine Learning

### The AlphaGo Approach

Two core ML models:
- **Value function** $$v(s)$$: approximates game outcome from state $$s$$
- **Policy function** $$p(a\vert s)$$: probability distribution over actions given state

**AlphaGo's modified UCT**:

$$a = \arg\max_{a \in A(s)} \left[ Q(s,a) + \frac{P(s,a)}{1 + N(s,a)} \right]$$

**Leaf evaluation**:

$$V(s_L) = (1-\lambda)\nu(s_L) + \lambda z_L$$

where $$\nu(s_L)$$ is the value network output, $$z_L$$ is the rollout result, and $$\lambda$$ is a mixing parameter.

<img src="{{ '/assets/notes/mcts-review/Pasted-image-20260411201947.png' | relative_url }}" alt="Pasted image 20260411201947" loading="lazy" style="max-width: 100%; height: auto;" />
### MCTS + Neural Networks

| Approach | Game/Domain | Key Innovation |
|----------|-------------|----------------|
| **AlphaGo/AlphaZero** | Go, Chess, Shogi | Policy + value deep CNNs with RL self-play |
| **MoHex-3HNN** | Hex | Three-head NN (policy, state-value, action-value) |
| **DeepEzo** | Hex | Policy trained from agents not using the policy |
| **Spear** | Task Scheduling | MCTS simulation replaced by deep RL policy |
| **MCTS as Trainer** | Atari, Pommerman | MCTS generates demonstrations for RL (UCTtoRegression, UCTtoClassification) |

#### 5.4 MCTS + Temporal Difference Learning

Replace UCT with a weighted average of classic UCT and TD estimates:

$$Q_{\text{TD-UCT} } = \gamma V + (1-\gamma)\left[Q + C\sqrt{\frac{\ln N(s)}{N(s,a)} }\right]$$

Three variants proposed (Vodopivec & Šter 2014):

| Variant | Key Feature | Performance |
|---------|-------------|-------------|
| **TD-UCT Single Backup** | Assumes future values converge to final playout value | Good |
| **TD-UCT Weighted Rewards** | $$\gamma=1$$: V fully replaces Q; keeps UCT exploration | Moderate |
| **TD-UCT Merged Bootstrapping** | Regular TD bootstrapping; state value updated from next state | **Best** (with AMAF, wins in all 4 test games) |

### MCTS + Evolutionary Methods (Section 6)

| Category | Method | Key Idea |
|----------|--------|----------|
| **Evolving Heuristics** | Genetic Programming for evaluation functions | Evolve board-evaluation functions offline, use in simulation phase |
| **Evolving Policies** | (1+1) ES, Knowledge-Based Fast-Evo MCTS | Optimize weight vectors for tree/default policies; dynamic feature extraction |
| **Evolving Tree Policy** | GP-evolved UCB replacement | Syntax tree encoding alternative selection formulas |
| **RHEA** | Rolling Horizon EA | Evolve action sequences; competitor to MCTS; neither is universally better |
| **Evolutionary MCTS** | EMCTS (Baier & Cowling 2018) | Nodes contain action sequences; edges are mutation operators; no rollouts |

### Applications Beyond Games (Section 7)

| Domain | Key Approaches | Notable Results |
|--------|---------------|-----------------|
| **Planning** | ASAP-UCT (state-action pair abstraction), PROST (action pruning + reward locks), Hierarchical MCTS, Dec-MCTS for multi-robot | ASAP-UCT significantly outperforms vanilla UCT; Dec-MCTS achieves 7% better score |
| **Security** | Mixed-UCT, Double Oracle UCT (O2UCT), AT-O2UCT | Finding Stackelberg Equilibria in multi-step security games via MC simulations |
| **Chemical Synthesis** | 3N-MCTS (3 neural networks guiding MCTS) | Published in *Nature*; plans retrosynthesis using 17,134 reaction rules |
| **Scheduling** | ProUCT (heuristic solvers + UCT), Spear (AlphaGo-inspired) | Spear reduces makespan by 20%; ProUCT handles risk-aware project scheduling |
| **Vehicle Routing** | Problem factorization (per-vehicle trees), macro-actions, UCT replacement | Combines Clarke-Wright initial solutions with MCTS-based iterative improvement |

## Parallelization (Section 8)

MCTS 的核心邏輯是迭代越多次統計量越可靠、決策越好。經典 MCTS 的四個階段是嚴格序列執行的，如果能平行化就能在相同時間內跑更多迭代。但 MCTS 的平行化跟一般的平行計算（如矩陣乘法）不同,,<mark>平行版本的結果不會跟序列版完全一樣</mark>。為了實現平行，必須犧牲一些東西（例如 UCT 的選擇公式不再是最優的），但只要整體能跑更多迭代，效能還是會提升。

### 三大基礎方法

**① Leaf Parallelization** (Cazenave & Jouandeau 2007)
- 只有一棵樹。Selection 跟 Expansion 照常序列執行，但 Simulation 階段從同一個新展開的節點同時跑 $$K$$ 個獨立 playout，結果加總/平均後 backpropagate
- 優點: 實作最簡單，每個新節點的估值更準確（$$K$$ 次 playout 的平均）
- 缺點: <mark>樹的成長速度不會加快</mark>，每次迭代還是只展開一個節點

**② Root Parallelization** (Chaslot et al. 2008)
- 開 $$K$$ 個完全獨立的 process，每個 process 自己建一棵完整的 MCTS 樹。在做出真正決策之前（同步點），把所有樹的第一層統計量（Q 值和拜訪次數）進行加權聚合
- 優點: 通訊開銷極小（只在同步點合併），聚合後統計量更有信心
- 缺點: 不會產生更深的樹（各子樹獨立探索，可能有大量重複搜索）

**③ Tree Parallelization** (Cazenave & Jouandeau 2008)
- 只有一棵共享的樹。$$K$$ 個 thread 同時在樹上執行完整的 MCTS 迭代（Selection → Expansion → Simulation → Backpropagation 全部由各 thread 獨立做）
- **Virtual Loss（虛擬損失）** 是關鍵 trick：當一個 thread 正在從某節點往下探索時，立刻給該節點記一個最差結果（假裝它輸了），降低其 UCT 分數，阻止其他 thread 也跑去同一個地方。等 playout 結束、真實結果回來後再修正
- 效果: 迫使不同 thread 探索不同分支 → 樹成長更快、更寬
- 缺點: 需要 locks/mutexes 做同步（展開節點和 backpropagation 是 critical section），有同步開銷
- ⚠️ Graf & Platzner (2015) 發現搭配動態策略更新（adaptive playout）會大幅降低平行效率，因為策略更新需要 global lock。16 threads 下 adaptive vs. static playouts 效能比只有 **68%**

| 方法 | 樹的數量 | 樹成長速度 | 估值品質 | 通訊成本 |
|------|---------|-----------|---------|---------|
| **Leaf** | 1 棵 | ❌ 不加快 | ✅ 更準確 | 零 |
| **Root** | $$K$$ 棵 | ❌ 不加快 | ✅ 統計量更有信心 | 極低 |
| **Tree** | 1 棵（共享） | ✅ 加快 | ⚠️ Virtual Loss 可能偏移 | 中-高（需 lock） |

#### 8.2 專用硬體加速

**GPU Parallelization** (Barriga et al. 2014)
- 在 GPU 上結合 Root + Leaf 平行化：GPU 建多棵獨立的樹（Root），每棵樹分配 32 的倍數個 GPU thread 做 Leaf 平行。在 Ataxx 遊戲中最佳配置達到 **77.5% 勝率**
- 挑戰: 遊戲模擬器必須在 GPU 上實現（很多複雜遊戲做不到），且無法控制 thread scheduling 和 CPU/GPU 通訊

**Intel Xeon Phi** (Mirsoleimani et al. 2015)
- 在 Xeon Phi（61 cores, 244 threads）上用 Tree Parallelization，把迭代拆成 chunks（grain size control）。比序列版快 **5.6 倍**

**MPPA** (Hufschmitt et al. 2015)
- 在 256 核心、16 clusters 的 MPPA chip 上嘗試了 propnet 平行化和 playout 平行化，後者效果較好但同步開銷仍偏高

#### 8.3 Lock-free Parallelization（無鎖平行化）

> Mirsoleimani et al. 2018

MCTS 中有兩個 critical moment 需要 lock：展開新節點 (Expansion) 和更新統計量 (Backpropagation)。Lock 能防止 deadlock 和 race condition，但會造成效能瓶頸。

解法是利用 C++ 的 memory model 和 `atomic` 關鍵字實現無鎖的平行 MCTS。`atomic` 操作保證對變數的讀寫是原子性的，不需要顯式加鎖。作者開源了函式庫，在 Hex 遊戲上驗證了 Tree 和 Root Parallelization 的有效性

#### 8.4 Root–Tree Hybrid（根-樹混合平行化）

> Świechowski & Mańdziuk 2016 (本篇作者)

<mark>擴展性最強的方法</mark>，三層架構：
- **Worker Nodes:** 每個 worker 維護自己的 MCTS 樹，內部用 Leaf Parallelization
- **Hub Nodes:** 從下層 workers 匯集統計量（Root Parallelization 的概念）
- **Master Node:** 最頂層做最終聚合

創新: **Limited Root–Tree Parallelization** 把搜索樹切成子樹分配給各 worker，但子樹之間不完全分離，保留一定的重疊（redundancy），因為 Root Parallelization 需要各 worker 對共同區域有估計才能有效聚合。在 9 個 GGP 遊戲中，混合方法效果超越單獨使用 Root 或 Tree

#### 8.5 遊戲特定 & Information Set 平行化

- **Hex 解棋** (Liang et al. 2015): Job-Level UCT 突破了 SPDFPN 演算法「thread 數量 ≤ CPU core 數量」的限制
- **Go** (Schaefers & Platzner 2014): UCT-Treesplit 用平行 transposition table + MPI 通訊，在 HPC cluster 上實現極佳的可擴展性
- **ISMCTS 平行化** (Sephton et al. 2014): 把三種基礎方法套用到 ISMCTS，在 Lords of War 紙牌遊戲上驗證 <mark>Root Parallelization 搭配 ISMCTS 效果最好</mark>,,因為 ISMCTS 本身每次迭代都會重新抽樣 determinization，Root 的獨立性天然吻合此特性

## Limitations & Thoughts

### Limitations of the Survey
- **Scope restricted to multi-agent focus**: Single-agent MCTS papers are included only if the authors judged the modifications to be generalizable to multi-agent settings , this introduces subjective selection bias.
- **Limited quantitative comparison**: Being a survey, it cannot provide head-to-head comparisons of all methods on the same benchmarks. The reader must consult individual papers for quantitative results.
- **Post-2012 focus**: Pre-2012 foundational works are summarized only briefly; readers must consult Browne et al. (2012) for the earlier literature.
- **No code or reproducibility**: The survey does not provide implementations or standardized benchmarks for comparing the reviewed methods.

### Relevance to DRL for DFJSP Research
- **MCTS as a complementary method to DRL**: The paper extensively documents how MCTS and deep RL can be hybridized , MCTS provides structured exploration while RL provides learned heuristics. This is directly relevant to scheduling problems like DFJSP where pure DRL may struggle with combinatorial explosion.
- **Scheduling section (7.4)**: The Spear system (Hu et al. 2019) demonstrates MCTS + deep RL for dependency-aware task scheduling, achieving 20% makespan reduction. This architecture , replacing MCTS simulation with a trained RL policy , could be adapted for DFJSP.
- **Action reduction techniques** (Section 3.1) are highly relevant for DFJSP, where the action space (job-machine assignments across operations) grows combinatorially. Options/macro-actions, progressive widening, and informed priors could tame this complexity.
- **MCTS as a trainer for RL** (Section 5.3.2): Using MCTS demonstrations to train DRL policies (as in Kartal et al. 2019) could provide a "safer" exploration strategy for DFJSP, where bad scheduling decisions are costly.
- **Dynamic environments**: The survey highlights MCTS modifications for dynamic/real-time settings (VSP-MCTS, RHEA), which align with the *dynamic* aspect of DFJSP where jobs arrive over time and machines may become unavailable.
- **Problem factorization** (Section 7.5): The per-vehicle tree approach in VRP is analogous to per-machine or per-job trees in DFJSP, potentially enabling more tractable search.
- **Open question**: Can MCTS be used as an **expert demonstrator** during DRL training for DFJSP, providing high-quality trajectories that bootstrap the RL agent's policy? The survey suggests this is a promising and underexplored direction for scheduling domains.

## Related Notes

- Monte Carlo Tree Search

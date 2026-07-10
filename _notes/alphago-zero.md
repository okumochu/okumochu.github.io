---
layout: page
title: "Mastering the Game of Go without Human Knowledge"
description: "PUCT math and why MCTS acts as a policy improvement operator."
category: research paper
order: 7
---

****

## Introduction

> Starting *tabula rasa*, our new program AlphaGo Zero achieved superhuman performance, winning 100:0 against the previously published, champion-defeating AlphaGo.

AlphaGo Zero 把整套系統收斂成一個簡單的迴圈: **MCTS 產生比 NN 更好的決策 : NN 學習去模仿這些決策 : 更好的 NN 再讓 MCTS 變更強**。論文最大的賣點是不用任何人類棋譜，只靠 self-play 跟這個 MCTS-policy-improvement 的閉環就達到超人表現。

對我來說重點不在 RL loss 本身 (就是 cross-entropy + MSE)，而在於 **MCTS 在這篇被當成 policy improvement operator 來用**，這個觀念是後續 AlphaZero / MuZero / 各種 plan-then-learn 系統的核心。

## 跟前代 AlphaGo 的對比

| Component       | AlphaGo (Fan/Lee)                            | AlphaGo Zero                                  |
| --------------- | -------------------------------------------- | --------------------------------------------- |
| 訓練資料            | 人類棋譜 supervised + self-play RL                | 純 self-play，from scratch                      |
| Network         | 分開的 policy net + value net (plain CNN)       | 單一 dual-head 網路 $$f_\theta(s) \to (p, v)$$ (ResNet) |
| Rollout         | Fast rollout policy 跑到終局，跟 value net 加權平均    | <mark>完全沒有 rollout</mark>，leaf 直接用 $$v$$                 |
| Selection bonus | UCT 變形 (rollout count + prior)               | PUCT (visit-count-based + prior)              |
| Tree policy 來源  | SL policy 提供 prior $$P$$                       | Self-play 出來的 $$p$$ 自己當 prior                   |

值得注意的是 <mark>拿掉 rollout 反而變強</mark>。原因是 rollout 的雜訊很大，而一個訓得夠好的 value network 能給更穩定的 leaf 估值: 多 simulation 一次比 rollout 一次便宜很多。

## MCTS in AlphaGo Zero

整體還是經典的 four-phase loop，但每一步都被 NN 改寫過。

### 樹節點儲存什麼

每條邊 $$(s,a)$$ 儲存四個量:
- $$N(s,a)$$: visit count
- $$W(s,a)$$: cumulative action-value (所有經過這條邊的 leaf 估值總和)
- $$Q(s,a) = W(s,a)/N(s,a)$$: mean action-value
- $$P(s,a)$$: prior probability，這條邊第一次被展開時由 NN 給出

### Phase 1: Selection (PUCT)

從 root 走到 leaf，每一步選
$$a_t = \arg\max_{a} \left[ Q(s_t, a) + U(s_t, a) \right], \quad U(s,a) = c_{\text{puct} } \cdot P(s,a) \cdot \frac{\sqrt{\sum_b N(s,b)} }{1 + N(s,a)}$$

直觀拆解:
1. $$Q(s,a)$$: exploitation，跟 UCT 一樣
2. $$P(s,a)$$: 由 NN 提供的 prior，<mark>這是跟原始 UCT 最大的差別</mark>。UCT 的 exploration term 只看訪問次數，PUCT 多了一個「神經網路覺得這步看起來不錯」的乘數
3. $$\sqrt{\sum_b N(s,b)} / (1+N(s,a))$$: 訪問次數還少時這項很大，鼓勵探索；隨 $$N(s,a)$$ 增長被壓下來
4. $$c_{\text{puct} }$$: 平衡 exploration/exploitation 的常數

PUCT 的好處是 **prior 強的 action 在早期就會被優先探索**，不會像純 UCT 那樣前期完全亂走。等訪問次數累積夠了，$$Q$$ 會主導，prior 的影響相對衰減。

### Phase 2 & 3: Expand and Evaluate (合併了，不再有獨立的 Simulation)

當 selection 走到 leaf $$s_L$$:
1. 把 $$s_L$$ 餵進 NN，拿到 $$(p, v) = f_\theta(s_L)$$
2. 對每個合法 action $$a$$，設 $$P(s_L, a) \leftarrow p_a$$ 並初始化 $$N=W=Q=0$$
3. <mark>Leaf value 直接用 \(v\)，不跑 rollout</mark>

這就是論文最關鍵的工程簡化: 把「Expansion + Simulation」兩個階段壓成 single forward pass。

### Phase 4: Backup

從 leaf 一路回到 root，沿路每條邊:

$$N(s,a) \leftarrow N(s,a) + 1, \quad W(s,a) \leftarrow W(s,a) + v, \quad Q(s,a) \leftarrow W(s,a)/N(s,a)$$

Two-player zero-sum game 要記得 $$v$$ 的正負號每步翻轉 (對手的好就是自己的壞)。

### Root Exploration: Dirichlet Noise

純 NN prior 在 self-play 早期很容易讓 search 永遠卡在同一條開局線上。解法是在 root 注入 Dirichlet 雜訊:

$$P(s_{\text{root} }, a) = (1-\varepsilon) p_a + \varepsilon \eta_a, \quad \eta \sim \text{Dir}(0.03)$$

論文用 $$\varepsilon = 0.25$$。重點是 <mark>只在 root 加</mark>，不在內部節點加: 內部要保持 NN 的決策性。Dirichlet 的 $$\alpha=0.03$$ 很小，會在合法 action 中隨機把一兩個 prior 推高，鼓勵試試平常不會走的開局。

### 從 MCTS 抽出實際走步

跑完 $$1600$$ 次 simulation 後，得到 root 各 action 的訪問次數 $$N(s_{\text{root} }, a)$$。最終走步機率:
$$\pi(a\vert s) = \frac{N(s,a)^{1/\tau} }{\sum_b N(s,b)^{1/\tau} }$$

- 對局前 30 步: $$\tau = 1$$，按 visit count 比例抽樣 (保持 self-play 多樣性)
- 之後: $$\tau \to 0$$，等價於 greedy 取 $$\arg\max_a N(s,a)$$
- <mark>為什麼用 visit count 而不是 \(Q\)</mark>: visit count 已經結合了 $$Q$$ 跟 $$P$$ 跟 exploration history，是更穩定的訊號；單看 $$Q$$ 容易被低訪問次數的 noisy 估值誤導

## MCTS 作為 Policy Improvement Operator

這是這篇真正觀念上的貢獻，值得單獨拉出來講。

NN 給的 prior $$p$$ 本身是一個 policy。跑完 MCTS 後得到的 $$\pi$$ (visit count distribution) 是另一個 policy。理論上跟實驗上都有: <mark>\(\pi\) 嚴格優於 \(p\)</mark>，因為 $$\pi$$ 多用了 lookahead 跟 value estimate。

訓練目標於是變成 **讓 NN 模仿 MCTS 的輸出**:

$$\mathcal{L} = (z - v)^2 - \pi^{\top} \log p + c\Vert \theta\Vert ^2$$

- $$(z-v)^2$$: value head 學最終勝負 (MSE)
- $$-\pi^{\top}\log p$$: policy head 學 MCTS 抽出的分佈 (cross-entropy)
- L2 正則化

從 RL 角度看這就是 **Policy Iteration**:
- Policy evaluation: NN 學 $$v \approx V^\pi$$
- Policy improvement: MCTS 拿 $$p$$ 做 prior 跑 search 得到更好的 $$\pi$$
- 然後把 $$p$$ 推向 $$\pi$$，循環

跟一般 actor-critic 的差別: improvement step 不是 gradient ascent 而是 <mark>explicit planning</mark>。這也是為什麼這套方法在有完美 simulator 的領域 (棋類、scheduling、化學合成) 特別強。

## 訓練細節 (摘要)

- $$4.9 \times 10^6$$ 場 self-play games
- 每步 $$1600$$ simulations，思考時間約 0.4 秒
- ResNet (20 或 40 個 residual blocks)，輸入是最近 8 步的棋盤 stack
- 訓練 72 小時打到超越 AlphaGo Lee；40 天打到超越 AlphaGo Master (89:11)
- 沒有任何 domain heuristic、沒有人類棋譜、沒有 rollout policy

## Limitations & Thoughts

### Limitations

- **依賴完美 simulator**: AlphaGo Zero 能 self-play 是因為圍棋的 transition function 已知且 deterministic。換到 imperfect information 或 stochastic transition (例如 scheduling 中有隨機到達工件) 就不能直接套，需要 MuZero 那種 learned model
- **計算成本極高**: 4.9M games × 1600 sims × ResNet forward pass，原文用了 TPU 群組。學術復現幾乎不可能在小規模重現完整訓練
- **沒有 ablation 區分 MCTS 跟 ResNet 的貢獻**: 同時換了 architecture (plain CNN → ResNet) 跟訓練 pipeline，paper 自己做的 ablation 顯示 ResNet + dual-head 比舊 architecture 強很多，但沒拆出 MCTS-no-rollout 跟 architecture 各自貢獻多少
- **單一 game type**: 雖然後續 AlphaZero 推廣到 chess/shogi，但這篇只報告圍棋

### Relevance to DFJSP / DRL for Scheduling

- **MCTS as policy improvement** 對 scheduling 直接適用: 在 DRL policy 上面包一層 MCTS，用 visit-count distribution 當訓練 target，比純 policy gradient 更穩定 (Spear 系統就是這個思路)
- **No-rollout MCTS** 在 scheduling 特別划算: scheduling 的 random rollout 信號很差 (makespan 受後續所有動作影響)，直接用 value network 截斷 simulation 更合理。這跟 [Monte Carlo Tree Search a review of recent modifications and applications]({{ '/notes/mcts-review/' | relative_url }}) 5.4 節提到的 early termination 路線一致
- **PUCT 公式**: 對 DFJSP 這種 action space 巨大但很多 action 一看就不會選的問題，PUCT 的 prior-driven exploration 比 vanilla UCT 適合 (UCT 一開始平均訪問太浪費)
- **Dirichlet noise at root**: 可以借來解 DRL 訓練早期 policy collapse 的問題
- **<mark>這篇的最大 takeaway</mark>**: 如果 DFJSP 有夠好的 simulator，跑 self-play + MCTS-guided imitation 比直接做 model-free RL 更有可能突破 plateau。是值得放在後續實驗計畫的方向

## Related Notes
- [Monte Carlo Tree Search a review of recent modifications and applications]({{ '/notes/mcts-review/' | relative_url }})

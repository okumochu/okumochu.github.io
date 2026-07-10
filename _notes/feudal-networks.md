---
layout: page
title: "FeUdal Networks for Hierarchical Reinforcement Learning"
description: "Manager/Worker architecture, transition policy gradient, and ablations explained."
category: research paper
order: 6
---

<img src="{{ '/assets/notes/feudal-networks/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

## Motivation

1. Sparse Reward with Long Term credit assignment
2. Store Memory for non-markovian environment

> **Note**
> 在 non-markovian 的環境下，沒辦法 fully depend on 當下的 state ，所以就需要過去的 state 也就是 memory
>
> 可以用一個 NN 去存知識，但是哪些知識在當下的 state 有用哪些知識有沒有用是需要 reward ，sparse reward make it more difficult

### Related Work

1. **Hierarchical Reinforcement Learning Foundations (options framework)**
   - **概念:** 高層會根據目前的環境選擇一個 option (policy-over-options)，讓底層執行 (sub policy)，並且會根據被指派的 option 一直執行 action 直到 terminate。然後再重新選一個 option
   - **舉例:** 主要目標是學會煮飯， sub goal 就會是走到 拿調味料 or 翻炒料理。每個 option 都會得到 pseudo reward 使得能走到 sub goal。那決定要用哪個 option 就會是高層的 RL 的決定
      - 之所以被稱為 sub-goal 或是 pseudo reward 是因為真正要學的是更高層次的目標 (學會排程 vs 學會排十個產品)
   - **問題:** sub-goal 和 pseudo-rewards 什麼時候 terminate 都要人工明確定義。其實很多 task 的 main goal 都沒有辦法很直覺的做拆解成 sub goal
2. **End-to-End Option Learning**
   - **概念:** 可以透過 policy gradient 同時訓練 policy-over-options (high level policy) and low level policy。這樣 high level policy 就會變成一個抽象的概念，不用 hand craft sub-goal 和 pseudo-reward 且也會自己學會何時 terminate
   - **問題:** 有可能 collapse 成 flat RL (等同於沒有階層，無法取得有 reusable 的 skill)
      - 出現只有一個活躍的 option 完成所有的事情
      - high level policy (policy-over-option) 每步都變動 option (solution: )
   - **解方:** option terminate 之後會給 mild penalty，使其 action 長度變長。然後再用 entropy bonus，使其 option 的型態多樣化
    

可以看到原本的方法是使用 option 來主導整個 sub task 的執行，這個選出來的 option 本身就是黑箱子你也沒有辦法確定這個 sub task 是否有意義

但是本篇研究提出的方法是是讓 high level policy 不要這麼主導而是**提供一個方向 (接下來幾步要向哪個方向改變狀態)，使得 low level policy 知道怎麼做。並且透過架構設計學習到有用的 sub-goal**

These improvements can be combined with hierarchical architectures like FuN for further gains .

1. **Non-Hierarchical (Flat) + Auxiliary Loss**
    
   還有一個方法是功能不干涉且互補的 (orthogonal)
    
   - FuN 主要提供功能是分解長期目標以及短期行動。也就是如何將漫長的決策過程拆成可學、可執行的小步驟 and 如何處理 long term reward, memory
   - 使用 Auxiliary Loss and Reward 來獎勵更多的探索 (e.g. pseudo-count based 每走到新的 state 就給 reward)
   - UNREAL agent 透過一個 Auxiliary Task 來使得 internal representation 更好

> **Note**
> 我們可以在 FuN 的 representation layer 上，加入 **UNREAL** 那樣的輔助頭 (pixel control、reward prediction 等)，讓 Manager 和 Worker 共用更強大的特徵
>
> 也可以一樣在 Worker 的強化學習損失中，加上 **pseudo-count** 內在獎勵，讓 Worker 在執行 Manager 指定的方向之外，仍然對未曾拜訪的狀態保持好奇

## Key Component

<img src="{{ '/assets/notes/feudal-networks/Screenshot_2025-05-08_at_3.37.58_PM.png' | relative_url }}" alt="Screenshot_2025-05-08_at_3.37.58_PM" loading="lazy" style="max-width: 100%; height: auto;" />

- **Manager**: operates at a lower temporal resolution, sets **abstract goals** (in latent space)
- **Worker**: acts **every step** to achieve these goals, produces primitive actions

Note:

- state → latent space, shared by manager and worker
- Manager is trained by “Transition Policy Gradient”
- Both Manger and Worker **are recurrent**

### Goal Embedding

- 就像前面提到的 state 會進入一個 latent space，再通過一個 recurrent network 產生出 $$g_t$$
- 把最近前 $$c$$ 期的 $$g$$ 加起來 $$\sum_{i=t-c}^{t} g_i$$ ，然後透過 **linear transform without bias $$\phi$$ 到低維度** → $$w_t$$
   - 這個  $$\phi$$ 是透過前面 worker 的 gradient 學來的
   - 其他的組件是透過 transition policy gradient
    
   > **Note**
   > 因為這個 $$\phi$$ 沒有 bias ($$\phi(x) = W x$$)，所以當 $$W =0$$  時，整項都會變成 0，進而使得 action 會是 zero vector。也就代表沒有 manager 的幫助 ($$W = 0$$) 時 worker 也做不了事情。所以 worker 必須要使用到 manager
    
   > **Note**
   > 以 Manger 來說每 $$c$$ 期才會換一個目標 $$g$$ (c x |g|)，這個 smoothing 的作用就是讓 Worker 轉換目標 ($$g_{old}$$ → $$g_{new}$$) 的時候很滑順
   > 不是直接 c 步都吃 $$g_{old}$$ 然後直接轉換成 $$g_{new}$$ ，而是設計一個 window，慢慢平均滑動過去，讓方向不會劇烈變動 (包括後面設計的都是 Linear Combination without bias) 
   >
   > old = [1, 1, 1, 1], new = [2, 2, 2, 2], c=4
   >
   >  中間那期的轉換就會是 (1+1+2+2) / 4 
    
- 最後，$$w_t$$ 會與 worker 的 embedding action 矩陣相乘直接通過 softmax 輸出
   - worker action embedding ($$U_t \in R^{k\times1}$$) $$\times$$ manager goal embedding ($$w_t \in R^{a\times k}$$ ) = $$a_t \in R^{a\times 1}$$

### Learning Process

> **Note**
> 其實整個框架都是 fully differentiable，所以可以用 policy gradient, TD-Learning 做訓練。但是如果這樣整個拿起來訓練就沒有 manager 的意義，整個中間內部設計都會變成 internal latent layer

所以

- Manger 提供的是 state 轉換的有利的方向，Manger 不直接參與每一步的選擇，而是每 c 步給予一次方向的調整。**後續會講解為什麼可以這樣更新**
    
   $$ \nabla g_t \;=\; A_t^M \,\nabla_{\theta}\,\text{cos sim}\bigl(s_{t+c} - s_t,\;g_t(\theta)\bigr) $$
    
   $$ A_t^M = R_t - V_t^M $$
    
- 上面就可以看出 manger 的輸出 $$g_t$$ 的更新公式是透過 advantage function  $$\times$$ 目標跟狀態 c 期差異的相似度
   - 只要 advantage function 是正的代表這次決策是比期望報酬好，這次造成的狀態轉移 $$s_{t+c} - s_{t}$$ 就是有利的，要讓 $$g_t$$ 學起來
- Worker 也會有一個 intrinsic reward 獎勵 worker 照著 manager 的 goal 做行動。那之所以是方向不是絕對位置，也是想要讓 worker 更能知道怎麼做行動，如果是絕對位置的話就沒有這麼 easy
    
   $$ \nabla \pi_t \;=\; A_t^{D}\,\nabla_{\theta}\,\log \pi\bigl(a_t \mid x_t; \theta\bigr) $$
    
   $$ A^D_t = R_t + \alpha R^I_t - V^D_t(x_t;\theta) $$
    
   $$ R^I_t \;=\; \frac{1}{c} \sum_{i=1}^{c} d_{\cos}\bigl(s_t - s_{t-i},\,g_{t-i}\bigr) $$
    
   > **Note**
   > 這邊有一個特殊的點在於 state is depend on $$\theta$$  ($$s_t = f_{Mspace}(z_t;\theta)$$) ，但是更新 worker 時沒有把它展開
   > 理由在於如果把這個 dependency 算進去，在做 back propagation 的時候，很有可能就會改變 representation 的那個 layer，而不是真正的改變 $$g_t(\theta)$$ 本身 (走捷徑)
    
   > **Note**
   > worker and manager 可以有不同的 discount factor，也就是說可以設定 worker 更 greedy，manager 遠視
    

### Transition Policy Gradients (Manager)

**A novel form of policy gradient** for the update rule of Manger

- High-level policy:  $$o_t = \mu(s_t; \theta)$$
   - 根據目前的 state 給選出一個 sub-policy: option (雖然說選但是這個 sub policy set 可以是 continuous)，這個 option 代表著 c 步的 action
- Transition distribution:  $$p(s_{t+c}\mid s_t,\,o_t)$$ based on 這個 option, state 經過了 c 步後的狀態機率分布為何
- 所以對於 Manager 的 policy 可以想像成一個 **transition policy** (transition distribution)，可以想像成 Manager 的 policy 直接選擇下一個 state $$s_{t+c}$$ 是什麼

$$ \pi^{TP}(s_{t+c}\mid s_t)  \;=\;  p\bigl(s_{t+c}\mid s_t,\;\mu(s_t;\theta)\bigr) $$

根據以上的轉換你就可以改寫原本的 policy gradient 的公式的 policy 的部分換成 transition policy。就可以看到原本的更新公式的每個 action 都要 chain rule 展開。如果把它想像成 one-step 的 transition policy 就不用了

$$ \nabla_\theta J = \mathbb{E}\bigl[(R_t - V(s_t))\,\nabla_\theta \log \pi(a_t \mid s_t)\bigr]. $$

$$ \nabla_\theta \pi^{TP}_t=\mathbb{E}\Bigl[(R_t - V(s_t))\;\nabla_\theta \log p\bigl(s_{t+c}\mid s_t,\;\mu(s_t;\theta)\bigr)\Bigr] $$

- 那我們這個 transition policy 是什麽呢？論文架設他是論文架設它是vMF distribution
   - 這個假設使得計算上比較方便

$$ p(s_{t+c}\mid s_t,o_t)\;\propto\;\exp\bigl(\kappa\;\cos(\,\Delta s,\;g_t)\bigr) $$

### Dilated LSTM (Manager)

note: Worker’s LSTM 是標準的 LSTM

- Dilated LSTM 是由  $$h = \{\,\hat h^1,\;\hat h^2,\;\dots,\;\hat h^r\}$$ 所組成
   - 每個 $$h$$ 被稱作 core ，可以想成記憶單元
- 每次更新時，會用以下會計算 $$t$$-step mod $$i$$ (e.g. 10 mod 3 = 1，所以只有 $$i$$ = 1 的記憶單元要更新)
   - 這樣代表每個記憶單元每 r 步才會做一次更新，剩下的都保留舊的資訊
- 輸出就像前面提到的會 average 前 c 步的結果才輸出

> **Note**
> 這樣的效果就是，因為更新不是整條一起更新，所以更能保留記憶

## Result

證明了這個架構是有用的，並且學習到有意義的 sub-goal and sub-policy

- Baseline model:
   - CNN  + last step action for state representation
   - LSTM for actions (316 hidden units)
   - BPTT K = 40, **400**
      - BPTT 是 RNN-base network 參數更新的方法，k 越大代表往回傳播的步數越多，造成的計算負擔越大
        
      > **Note**
      > 之所以 flat LSTM 是 40，但 proposed version 是 400 是因為本研究的 r = 10，所以根據計算量來說 400 / 10 才會相同
        
   - A3C algorithm
- 每 1M step 算成一個 epoch
- 每個方法都跑 100 次實驗，每個實驗會設置一個超參數 (從各自適合的區間抽出)

> **Note**
> This choice means that FuN and the LSTM baseline to have
> roughly the same number of total parameters

### Montezuma’s Revenge

陷阱多、獎勵稀疏為特色

- Manager $$\gamma$$ = 0.999 longer horizon
- Worker and Baseline $$\gamma$$ = 0.99

依照架構設計上 Manger 會給出一個方向 $$g$$ 並且在 worker 會未來 c 步跟著這個目標執行。當找出那些**使 $$\cos(s_{t+c}-s_t, g)$$ 最大**的狀態，這些狀態就是最能實現這個目標向量的時刻

下圖可以看到，如果取出那些 $$\cos(s_{t+c}-s_t, g)$$ 最大化的遊戲畫面，發現它們恰好對應到梯子旁、鑰匙處等關鍵位置，到達這些關鍵位置**是在過往研究中被手動設定的子目標**，但是也就是 FuN **自動學到的**

<img src="{{ '/assets/notes/feudal-networks/Screenshot_2025-05-11_at_10.33.47_PM.png' | relative_url }}" alt="Screenshot_2025-05-11_at_10.33.47_PM" loading="lazy" style="max-width: 100%; height: auto;" />

- **LSTM**: needs **>300 epochs** just to pick up the key and open the first door (score ≈400). Thereafter it **stalls** until ~900 epochs before making any further progress.
- **FuN**: solves that first room in **<200 epochs**, then **immediately** explores deeper rooms, eventually reaching scores of **~2600**.

### ATARI

note: Worker fixed at 0.95

<img src="{{ '/assets/notes/feudal-networks/Screenshot_2025-05-11_at_10.58.15_PM.png' | relative_url }}" alt="Screenshot_2025-05-11_at_10.58.15_PM" loading="lazy" style="max-width: 100%; height: auto;" />

- **FuN is strong** when you need multi-hundred-step credit assignment or deep exploration.
- **Greedy, short-horizon games** can actually be easier for a flat LSTM with a lower discount.

**Option-critic architecture** 

在過去的研究中 Option-Critic 架構 flat DQN，並且效果差不多。本研究使用的是更強的 baseline，且都 outperform，因此贏過此架構應該是毫無懸念

### Memory in Labyrinth

**memory‐and‐long‐horizon tasks** built in DeepMind Lab

Many tasks require you to **remember what you saw** earlier,plain pixel inputs are insufficient for optimal decisions unless you carry over those past observations in memory

<img src="{{ '/assets/notes/feudal-networks/Screenshot_2025-05-11_at_11.16.38_PM.png' | relative_url }}" alt="Screenshot_2025-05-11_at_11.16.38_PM" loading="lazy" style="max-width: 100%; height: auto;" />

### Ablation Study

recap this paper proposed

1. Transition Policy Gradient: 用 predicted transition distribution 的 policy gradient 做訓練，而不是直接用 worker 的 reward
2. relative sub-goal: 給予目標 $$g$$ 時不是給予絕對位置，而是給予一個方向
3. Intrinsic Reward for worker: 同時接受外部獎勵，以及跟隨 goal 的指引的內部獎勵

| **non Feudal FuN** | Manager 的目標 
$$g$$ 透過 **Worker 的 reward 訓練**，且 **不使用內部獎勵**。類似 Option-Critic |
| --- | --- |
| **Manager via PG** | Manager 的目標 $$g$$ 是透過標準 policy gradient 做訓練，會輸出**高斯分佈的平均來 generate  $$g_t\sim \mathcal{N}(\mu_\theta(s_t),\Sigma$$)** (as if continuous control problem)  |
| **Absolute Goals** | goal 從 相對方向 改為 絕對位置。Worker 的 intrinsic reward也改為到達該絕對位置的距離 |
| **Pure Feudal** | 捨棄 worker 的 reward，只用 **intrinsic reward** 訓練 Worker
Manager 仍用 transition policy gradient，$$g$$ 仍然是相對方向 |

<img src="{{ '/assets/notes/feudal-networks/Screenshot_2025-05-11_at_11.27.05_PM.png' | relative_url }}" alt="Screenshot_2025-05-11_at_11.27.05_PM" loading="lazy" style="max-width: 100%; height: auto;" />

### ATARI action repeat transfer

這個實驗很有趣，是在說 action 的執行頻率，之所以會中這個變因是因為 ATARI 是 pixel 遊戲，所以如果 action 頻率很高，角色可能就會一直抽動

所以通常在 ATARI 實驗上每一個 action 會被重複四次，使其移動能更滑順

那如果我們把訓練在重複四次的 policy 轉移到 重複一次的環境下呢？

調整一下設定

- $$\gamma$$ 需要調整，因為現在做一步的效果變短了
- dilated LSTM 的 
$$r$$ (決定多久更新一次的超參數) 也需要調整
   - e.g. 在每一個 action repeat 4 times 原本每 10 步 (r = 10) 更新一次，現在會變成每 40 步更新一次。因為原本的 10 步其實就是 40 步
- Manager 的 horizon 
$$c$$，跟以上同理需要變成 4 倍

最後調整完，那兩個已經在 action repeat = 4 的 baseline and FuN，並且拿來 fine tune 在這個新環境上

<img src="{{ '/assets/notes/feudal-networks/Screenshot_2025-05-12_at_9.54.29_AM.png' | relative_url }}" alt="Screenshot_2025-05-12_at_9.54.29_AM" loading="lazy" style="max-width: 100%; height: auto;" />

結果其實可看到 LSTM 在 policy 轉移上其實沒有提升太多 (除了第二個遊戲)

> **Note**
> FuN 做的比較好的原因是因為 其實 high level policy 在做的事情是給予一個方向，然後 worker 的工作就是想辦法達到那個目標。這樣子的架構對於 action frequency 不太敏感。 因為重點在於持續往目標走，而不是多少步完成工作

## Discussion and Future Work

Generalization 一直都是很重要的問題。比起硬背每步 action 要怎麼做，學習到有意義的 skill 並且重複利用是其中一個很有效果的解法

還有一個很重要的是 sub-goal 是很難一個一個定義的，像是排程有很多 heuristic rule 但是你確定你可以 define 所有的 rule 嗎？所以有一個架構能夠自己學習有意義的 sub-goal 並且指引 low-level policy 是很有用的

> FuN clearly separates the module that discovers and sets sub-goals from the module that generates the behavior


### Future Work

- **Deeper hierarchies**: more than two levels, each operating at its own time scale (e.g. goals every 1 step, 10 steps, 100 steps)
- **Scale to truly large, sparse, partially‐observable environments**: e.g. open‐world navigation, robotics, multi‐room mazes
- **Transfer & multitask learning**:
   - Once discovered, those **behavioral primitives** (transition policies) can be **reused** to jump‐start new tasks
   - Likewise, a trained Manager can be **transferred** to a new embodiment (robot body, frame‐rate) by retraining only the Worker

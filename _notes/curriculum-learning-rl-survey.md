---
layout: page
title: "Curriculum Learning for Reinforcement Learning Domains: A Framework and Survey"
description: "Section-by-section summary of the curriculum learning framework and its design dimensions."
category: research paper
order: 2
---

> Narvekar, S., Peng, B., Leonetti, M., Sinapov, J., Taylor, M. E., & Stone, P. (2020). Curriculum learning for reinforcement learning domains: A framework and survey. Journal of Machine Learning Research, 21(181), 1-50.


## Introduction

- 在人類的學習中，循序漸進是非常重要的。這對於 agent 來說也是一樣，過去的實驗證明說如果能循序漸進的學習任務 → faster convergence, better performance
   - 例如: 要讓一個人學會下一盤棋，可以先從多個 sub-game 開始。每次都學會一點事情像是國王要活著，棋子要如何移動等
        
      <img src="{{ '/assets/notes/curriculum-learning-rl-survey/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
- 從遠觀來說，CL 就像是運用過去的知識來學習更難的 task，這其實就是跟 transfer learning 要做的事情是有重疊的，但是比較像是 few-shot 來做這件事情
- 最基礎的角度定義 Curriculum = individual experience sample.
    
   > One task can build upon knowledge gained from multiple source tasks, just as courses in human education can build off of multiple prerequisites.

- 可以把課程想像成一個 DAG (directed acyclic graph)，所有的路徑的節點都是為了後面匯流的節點的經驗，最後就會匯流到終點 (final task)
   - 這個 DAG 可以的邊可以在訓練前就定義好，也可以是動態調整

## Transfer Learning

> In the standard reinforcement learning setting, an agent usually starts with a random policy,
and directly attempts to learn an optimal policy for the target task. When the target task is
difficult, for example due to adversarial agents, poor state representation, or sparse reward
signals, learning can be very slow.

- 所以會用 transfer learning leverage 過去訓練好的結果，這個結果不限於任何形式 e.g. samples (從舊的 task 找關鍵的 data point for new agent), option, policy, value network, model
   - 這些 transfer 的方法都會有一些假設像是 shared  state, action space, transition, reward function 等。總之要選哪種方法跟 source task 和 target task 有很大的關係，確定好關係後是至可以有multiple source tasks transfer to one task
- 如何判斷 transfer learning 的成效: 通常比較 transfer learning 和 from-scratch 的 training curve (expected return)
   - *Time to threshold* (比較速度), *Asymptotic performance* (比較最終收斂), *Jump start* (比較一開始的收斂情況), *Total reward* (比較在一段時間內的 total reward)
   - 這邊也有一個很重要的議題就是要不要把 transfer learning 花的時間算進去
      - 有算進去叫做 strong transfer setting，否則叫做 weak transfer setting
        
      <img src="{{ '/assets/notes/curriculum-learning-rl-survey/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
        

## Curriculum Learning Method

<img src="{{ '/assets/notes/curriculum-learning-rl-survey/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

**TL;DR** 不同的任務會構成一個一個節點，然後這些節點上的邊的方向就代表 target task 需要的 source experience
Note: power set 就是指**是指某個集合的**所有子集所組成的集合，包括空集合和它自己。 e.g. $$D^{\mathcal{T} } = \{a, b\}$$
,  $$\mathcal{P}(D^{\mathcal{T} }) = \{\ \varnothing,\ \{a\},\ \{b\},\ \{a, b\} \ \}$$

- 通常 sample level experience 的定義上比較難去實作，所以就有之後的簡化的定義
   - **Single-task Curriculum**
      - 所有樣本都來自同一個任務，這樣問題就變成在這個 task 下要怎麼排序每一個 sample
      - 典型出現在 experience replay 類方法裡
   - **Task-level Curriculum**
      - 每個節點代表一個任務的樣本集合，通常會從前面的任務學完才會學下一個任務。但是，這不代表前面的 task node 訓練過後就不能再訓練了，因為一個 node 可以有多條 vertex，若是需要前面的 task 還是會被訓練
      - 挑任務的先後次序，會影響之後行為策略收集到的樣本分布
   - **Sequence Curriculum**
      - 最後這個最常見也最好實作，就是照順序訓練像是一條 chain 一樣 (每個 node 只有一個邊)
   - Mixed: 這些方法也可以混合 e.g. Sequence + Task 每階段學習一群 tasks
- 最後很重要的就是多久需要換下一個節點
   - 通常實務上，當收斂時或是固定 epochs 數量
   - 不一定要每個 node 都要收斂，因為說不定 target task 還需要別的 node 的訓練資料才會成功

### Curriculum Learning

> Curriculum learning is a methodology to **optimize** the **order** in which **experience** is accumulated by the agent, so as to **increase** **performance** or **training speed** on a set of **final tasks**. 
Through generalization, knowledge acquired quickly in simple tasks can be **leveraged to 
reduce the exploration** of more complex tasks. 
In the most general case, where the agent can acquire experience from multiple intermediate tasks that differ from the final MDP, there are 3 key elements to this method

- **Task Generation:** 這個步驟的重點就是要生成出中介任務 (node)，讓 agent 順利過度到 final task。可以由人工定義中介任務或者是根據 agent 的表現動態生成
- **Sequencing:** 有了中介任務之後就是要找出順序 (vertex)，同樣的這個步驟可以人工定義或是根據 agent 狀況去自動生成 (**本篇 survey 會著重在這個步驟**)
- **Transfer Learning:** 這個步驟主要是負責處理不同 task 之間可能會出現的 MDP 的不同 (state/action space, reward function, or transition function from the final task)。這個步驟在 transfer learning 的文獻中有很好的討論了，故本篇 survey 不會花太多筆墨討論
- Evaluation 的方法就跟 Transfer Learning 相同，就是比較有 CL 和沒有 CL
   - 至於 strong, weak transfer 除了考慮時間的成本之外，還有建立 curriculum 本身的成本。但是目前很多步驟都是人工定義的所以成本很難量化
   - 所以就會變成 case by case 計算出人花費的成本以及 data collection 的成本判斷否值得
      - 有些目標可能是達到某個 performance threshold 就可以了

### Dimensions of Categorization (for communication)

1. **Intermediate task generation**
   - **Target:** 其實沒有中介任務，課程就是直接在目標任務上排序資料 e.g. 在目標任務經驗回放中，先學簡單樣本
   - **Automatic:** 由演算法自動生成中介任務，可能透過環境參數隨機化、generator 或對抗式方法自動合成
   - **Domain Experts:** 由領域專家根據知識設計 e.g. 製造業專家依生產瓶頸設計中介場景
   - **Naïve Users:** 非專家使用者手動設定 e.g. 遊戲開發者調一些難度參數，沒有特別的最優化過程
2. **Curriculum representation**
   - **Single:** 單任務情境，只有目標任務，但你對樣本排序
   - **Sequence:** 線性序列，每個節點最多一個前驅、一個後繼
   - **Graph:** 一般的 DAG，可以分支、合併，允許多路徑達到目標
3. **Transfer method**
   - **Policies:** 直接把上一任務學到的策略權重初始化給下一任務。
   - **Value Function:** 只遷移 value network
   - **Task Model:** both policy and value function
   - **Partial Policies:** e.g. options、skills
   - **Shaping Reward:** 用前一任務的知識設計額外獎勵引導下一任務
   - **Other / No Transfer**
4. **Curriculum sequencer**
   - **Automatic:** 演算法自動根據表現或某種啟發式調整順序。
   - **Domain Experts:** 由專家設計排序。
   - **Naïve Users:** 由非專家手動排序。
5. **Curriculum adaptivity (avoid sticking in some task)**
   - **Static:** 一次性設計好，不會變。
   - **Adaptive:** 根據學習進度或表現動態調整課程路徑。
6. **Evaluation metric**
    
   Performance metrics
    
   - **Jump start:** 初始表現好壞
   - **Time-to-threshold:** 達到一定表現門檻所需時間
   - **Asymptotic Performance:** 長期收斂表現
   - **Total Reward Ratio:** 訓練至某時間點的累計回饋比
    
   Baseline of performance
    
   - **Weak Transfer:** 只計算目標任務的學習成本（忽略中介任務與課程設計成本）
   - **Strong Transfer:** 包含中介任務與課程設計的全部成本
7. **Application area…**

## Task generation

> All the generated tasks should be relevant to the final task(s) and avoid negative transfer, where
using a task for transfer hurts performance
這個步驟研究的人是相對少，可以說是 CL 的瓶頸，因為


其實這個步驟是相對少探討到的部分，主要的 work 都假設當 task 可以被參數化表示，也就是說不同的參數就可以直接決定不同的 task

- 基於領域參數的規則生成: 然後直接在參數做任務簡化、初始化在比較簡單的地方、錯誤回溯
- 物件導向 MDP (OO-MDP): 將最終任務的子集 = 一個一個物件，然後隨機抽取組成環境
- Powerplay: 自動生成目前最簡單但尚未熟悉的環境，把生成跟學習綁在一起

## Task Sequencing

> Given a set of tasks, or samples from them, the goal of sequencing is to **order them in a way that facilitates learning**. Many different sequencing methods exist, each with their own set of assumptions. One of the fundamental assumptions of curriculum learning is that we can configure the environment to create different tasks


### Sample Sequencing

> we consider methods that reorder samples from the final task, but do not explicitly
change the domain itself


也就是說這類方法刻意地**安排這些 sample 訓練的順序** (而不是 random shuffle)，並且這些 sample 都是來自 final task

過去的研究證實，如果把這些 experience 由簡單到困難可以增加收斂速度以及泛化能力 

在 Q-learning 抽取 replay buffer 的資料的時候就能應用個概念，可以不需要隨機抽取。而是根據某個定義的指標 (有用程度、困難程度) 來抽取，下面介紹一些可用的指標

- **Prioritized Experience Replay (PER)**: 通常比較大的 TD-error 就代表這個模型對於個 state 比較不熟悉，在學習上的重要程度就越大，所以多學習這些 TD-error 大的的例子
- **Complexity Index (CI) function:** 運用一個 self-paced prioritized function (選一個最適合難度的) + coverage penalty function (不要一直抽取之前已經抽取過的)。成效比 PER 好，但是需要很大程度的精心為每個 case 做設計
- **ScreenerNet:** 避免 priority 由人工定義，會有第二個模型根據 TD-error 來判斷每個 sample 的 priority weight

除了根據 sample 的重要程度排序訓練之外，還有一種是直接 reconstruct 整個 training process based on the experience。會需要這樣的方法是因為通常有學踩過的 state 是相對於沒有踩過的 state 更好到達的，這件事情在 sparse reward 下更加明顯 (探索問題)

- **Hindsight Experience Replay (HER):** 轉換這些 undesired outcome → setting  outcome，並學習一個 Universal Value Function Approximator (UVFA) 。透過這個 policy 雖然可能沒有到達到 $$g$$，但是會到達另一個 undesired termination state $$s_T$$，雖然這個失敗的 trajectory 對於真正的目標 $$g$$ 沒有太大的幫助，但是如果改目標 $$g$$ = $$s_T$$ 就會對整個學習有幫助了
   - hindsight 事後諸葛
   - Universal Value Function Approximator = $$V_\pi(s,\text{goal})$$
- **Curriculum-guided HER (CHER):** 因為 replay buffer sample  goal evenly，所以用了兩個指標 curiosity (產生更多樣的 goal) 和 proximity (goal 跟真正的 goal 有多遠) 來抽取 replay buffer 的 goal。這樣就能有 diverse goal 並且慢慢網真正的 goal 靠近
- **Episodic backward update (EBU):** 這個算是一個加速強化學習的一個 trick，especially for delay reward case。這個就是讓 replay buffer 在抽的時候直接抽一整個 episode 的量的 batch size (這樣才會抽到有 reward 的地方)，並且從最後一步更新回去。從最後一部的原因是最後一步 (terminate step) 可以比較準的直接更新，然後倒數第二步就可以用更準的 Q function 來更新。當然這會造成 Cumulative overestimation errors，所以會需要一個 hyper parameter $$\beta$$ 來控制一下

### CO-Learning

這個類型的方法就是有多個 agent 在同一個環境 self-play (e.g. AlphaGo)，這種方式可以像像就是因為對手會從若到強，所以 implicitly 算是 CL

- **Asymmetric self-play:** 會有一個 teacher (proposed task) 和 student (do task)。Student 的 goal 就是花最少步數完成任務；Teacher 的目標就是最大化自己做起來的步數以及學生真實完成的步數。這樣當學生進步時自己也可以提出更難的題目
    
   > **Note**
   > 老師要覺得這個任務很簡單，但學生要實際做起來很難。因為這代表問題本身可能很簡單，但是學生太菜所以才花這個多部署
   > 這樣的方式就是 A curriculum of intrinsic exploration 而不是完完全依靠外在獎勵的探索
    
   - Teacher proposed a goal (如果環境是 resettable): student 從 initial state 試圖到達 final state
   - Teacher proposed a starter point (如果環境是 reversible): student final state 試圖走回 initial point

### Reward and Initial/Terminal State Distribution Changes

> Keep the state and action spaces the same, as well as the environment dynamics, but allow the **reward function and initial/terminal state distributions to vary**

- **Learning from easy mission**: 就先從 goal state 近的地方當作 initial state，然後每個 task 就是多一步 step size (越來越 long horizon 的 task)
- 剛剛那個比較像是 reverse expansion，也有 forward expansion，就是生成多個目標 $$g$$。就會有一個 Setter 和 Solver 和 Judge
   - Setter 要最大化 goal validity (achievable)、goal feasibility (solved by current skill)、goal coverage (choose diverse skill)
   - Judge 用來 estimate reward of solver when the goal is achieved
- Scheduler (meta-agent): 這個方法是一個 HRL 的方法，首先會有很多 intention (sub-goal)，下層 RL agent 會先學會如何達到這些 intention，然後再用 scheduler 排序這些 intention 來達到最終的目標。
    
   > Auxiliary tasks are defined to be tasks where the state, action, and transition function are the same as the original MDP, but where the reward function is different

- **Self-Adaptive Goal Generation Robust Intelligent Adaptive Curiosity (SAGG-RIAC):** 主要就是透過下面兩個數值， interest 大的區域的 goal 更有機會被訓練到。這樣就能快速學習到，但是因為這樣抽取 goal 出來在訓練上不一定會直直的往 target task 前進
   - Competence: 實際最終狀態以及目標狀態的距離
   - Interest: competence 在訓練中變化的速率 (越大代表越能快速掌握)
- **Training agent for first-person shooter game with actor-critic curriculum learning:** 這篇是教導 RL agent 玩毀滅戰士 (然後是 partially observable)
   - 一開始所有的地圖、reward 都是固定的，接下來就是改變一些不同的初始環境
        
      > Each curriculum consists of a sequence of progressively more complex environments with varying domain parameters (e.g., the movement speed or initial health of the agent)

   - 用剛剛的方式學完 (capable initial task model) 之後，算是第一階段結束，接下來地圖就會變動了，reward 也會變動。這邊使用的也是 adaptive curriculum learning strategy，所以會隨著模型的能力變強，課程會變得更加的困難

### No restrictions

這個部分使用 intermediate task (中間任務，也就是墊腳石的感覺) 來作為課程，並且這些中間任務沒有任何 MDP 的限制，也就是說中間任務可以在狀態空間、動作空間、轉移函數或獎勵函數等任何方面都與最終目標任務不同

**MDP-based Sequencing**

這個部分直接把 sequencing 的這個問題想成一個 MDP 來解決 

- **Autonomous Task Sequencing for Customized Curriculum Design in Reinforcement Learning Curriculum MDP (CMDP):** 根據目前 student policy 當作 state，下一個任務作為 action，學會課程花費的時間 (number of epochs) 作為 reward。當 student update policy 的時後 state 就會改變
- **Teacher–student curriculum learning:** 將此問題框架化為一個 partial observation MDP (POMDP)，無法直接存取 student’s parameters。其獎勵是學生在某任務上的分數相較於上次訓練該任務時的變化量 ，目標是最大化所有任務的總性能 。他們使用的啟發式方法是選擇學習曲線上斜率絕對值最高的任務

**Combinatorial Optimization and Search**

想像成一個組合最佳化問題，目標值就是之前有介紹過的評量方法 ( e.g. time to threshold, maximum return (asymptotic performance), and cumulative return )

可想而知方法就是 exact method, heuristic, meta-heuristic (trajectory-based, population-based)

- **Curriculum learning for cumulative return maximization:** 這篇論文先從所有 pair task (source ~ target) 的 tranferability，也就是用一個 simulator estimate cumulative reward 看是否從這個 task 到下一個 task 會提升整體能力，然後再根據這樣的數值做排序，然後就會得到一個 sequence。這個方法比較屬於 Heuristic 的部分
- **Faster reinforcement learning using active simulators:** 這篇一樣有一個 simulator estimate cumulative reward from any source task to any target task，但是這個是 online version
   1. 把所有的 task 先跑一個固定的步數 reward 高的先學 (當作 source)，因為 reward 高得比較有可能有好的 progress
   2. 用剛剛的 simulator estimate transferability matrix，直接 greedy 選出一條  learning chain
   3. 用剛剛每個 task 的 feature 訓練剛剛的 simulator (regression) 讓他能產出更好的  transferability matrix
- **A gray-box approach for curriculum learning:** 外層使用 MILP 做 scheduling (task sequencing)，內層使用原本的 RL 來完成 task。(這個我沒有很懂…)

**Graph-Based Sequencing**

這個方法把問題想成一張圖，task = node / sequence = edge

> Existing work has relied on heuristics and additional domain information to determine
how to connect different task nodes in the graph

- **Automatic curriculum graph generation for reinforcement learning agents:** 所有的 task 都是已知並且 feature of each task 是根據 domain knowledge (PAC-Man: number of ghost and type of maze)
   1. 先把 task feature vector 轉換成 binary feature vector (只標示哪些特徵存在(非零)，哪些不存在)，再根據這個二元向量，將擁有相似特徵組合的任務分到同一群
   2. Sub task 會各自行程 sub-graph 並且相互做連接，用 Heuristic 的方式 (transfer potential) 決定連接的方式
      - Transfer potential 正比於 Applicability / Cost
      - Applicability: ****source task 中學到的 value function，有多少 state 可以直接應用於 target task。適用性越高，表示知識轉移的效果越好。
      - Cost: ****學習源任務所需的成本，這裡用其 **size of its state space** 來近似。狀態空間越小，代表任務越簡單，學習成本越低
   3. 最後一步就是把剛剛的 sub-graph 連接起來，從特徵比較少的地方連接到特徵比較多的地方。這樣的方式可以確保從簡單學到難
- **Object-oriented curriculum generation for reinforcement learning:** 這個方法跟上面的方法類似，不同的是他把每個 state 視為一個物件，並且一個 task 是由這些物件所構成
   - Task Generation: 可以從最終的目標任務中，選擇一個較小的**物件子集**，來自動生成一個更簡單的源任務。
   - Sequencing: ****同樣採用了 Transfer potential 概念，但進行了調整: 它不是計算可應用的狀態數量，而是直接比較源任務和目標任務之間的物件集合 (因為越小的 object set 就代表越簡單的任務)
   - Limitation: ****雖然任務的排序是自動化的，但仍然需要人為介入 (human input)，以確保自動創造出來的那些簡化任務是可以被解決的 (solvable)。

### Auxiliary Problems

> **How long to spend on each intermediate task in the curriculum.** 
Most existing work trains on intermediate tasks until performance plateaus. However, as we mentioned previously, Narvekar and Stone (2019) showed that this is unnecessary, and that better results can be obtained by training for a few episodes, and reselecting or changing tasks dynamically as needed.

- **Curriculum learning with a progression function:** 這篇論文提到應該使用 progression function，這個 function 能夠根據難度引導這個 intermediate task 要訓練多久
   - 這個方法需要一個從 task generator estimate 問題的難度到 [0, 1] (1 就是 final task)
   - 有了這個 progression function 就有兩種使用方式 (根據難度線性或是指數反應)
      1. Fix progression: 訓練前就固定了
      2. Adaptive progression: 訓練中還可以調整 (根據模型的能力，也就是 reward 動態調整目前模型認為的難度)

### Human-in-the-Loop Curriculum Generation

> Humans may be able to design good curricula by considering which intermediate tasks are “too easy” or “too hard,” given the learner’s current ability to learn, similar to how humans are taught with the zone of proximal development


這個部分就是很多實例顯示課程 (intermediate task) 若有專家的 know how，將能大幅度的史訓練加速或是取得更好的表現。 e.g. 將一個轉換成多個技能，並且把相關的技能放在一起學習 (overlapping
layered learning)

<img src="{{ '/assets/notes/curriculum-learning-rl-survey/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

- 專家能設計出高品質的課程，但過程如同黑盒子 (intuition)
- 而非專家的行為模式和 trajectory 更具通用性的教學原則

> **Note**
> 其中有一個很有趣的例子就是讓人類去玩遊戲，然後收集那個遊戲的 checkpoint，然後就讓 RL agent 在這些 checkpoint 當作 initial point 開始學習 (我猜就是離終點最近的開始學，這樣問題就會越來越難) 

## Knowledge Transfer

<img src="{{ '/assets/notes/curriculum-learning-rl-survey/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> 這個部分就是在探討當 CL 在改變環境下，如何將知識轉移

> Transferred knowledge can be **low-level**, such as an entire policy, a value function, a full task model, or some training instances, which can be **directly used to initialize the learner** in the target task. The knowledge can also be **high-level**, such as partial policies or options, skills, shaping rewards, or subtask definitions. This type of information may not fully initialize the learner in the target task, but it could be used to **guide the agent’s learning process** in the target task. In this subsection, we discuss different transfer learning approaches used in curricula.


### Transfer low-level policies or value functions

- **Catastrophic Forgetting**: 當換到下一個 task 的時候完全忘記前一個學習過的知識，在學習兩個 orthogonal 的 task (two independent skills) 的時候就會需要注意
   - Progressive neural networks: 這是一種解法，一個 column 一個 task 做訓練，其他就都 freeze 住，這樣可以有效處理 Catastrophic Forgetting。限制在於 task 的數量會需要事先知道同時也會因為 task 增加模型也會增加
- **State and action spaces differ between tasks**
   - Transfer higher-level knowledge across tasks (option, skills, macro-actions

## Ch5, Ch6 Summary by Gemini

### **第一部分：課程學習的情境定位 (第五章內容解析)**

在我們的論文中，確立課程學習的獨特定位至關重要。許多研究方法都旨在提升學習效率，但它們的 underlying assumptions 和目標各不相同。第五章的目的便是釐清這些概念，確保我們對課程學習的討論建立在一個清晰的比較基礎之上。

### **1.1. 在強化學習諸多範式中的定位**

強化學習（Reinforcement Learning, RL）領域的一大核心挑戰是解決「樣本複雜度」（sample complexity）問題 1。許多方法都致力於此，但課程學習的切入點有其獨特性。

- **與其他樣本效率提升方法的區別**：模仿學習（imitation learning）、離策略學習（off-policy learning）及基於模型的學習（model-based approaches）等方法，各自依賴於不同的前提，例如人類專家的示範、足夠覆蓋率的行為策略數據，或是一個可供規劃的精確環境模型。相比之下，我們的課程學習方法建立在一個不同的核心假設之上。我們認為，「主要的假設是，環境可以被配置以創建不同的子任務，並且代理（agent）更容易在這些子任務中自行發現可重用的知識片段，這些知識可用於解決更具挑戰性的任務」。這意味著我們不依賴外部示範，而是透過主動地、有結構地重塑學習環境本身來加速學習進程。
- **與多任務學習相關範式的辨析**：多任務學習（multitask learning）、終身學習（lifelong learning）、主動學習（active learning）與元學習（meta-learning）等範式，雖然都涉及處理一系列任務，但其核心目標與課程學習存在根本差異。多任務與終身學習的目標通常是「同時優化所有 n 個任務的表現」，或是優化所有遇到任務的整體性能 。元學習則旨在訓練代理快速適應新任務的能力 。然而，我們論文中課程學習的獨特之處在於其明確的導向性。正如我們所強調的，「與這些工作相比，課程學習的顯著特點是，在課程學習中，我們完全控制任務被選擇的順序」。**更重要的是，其最終目的並非泛化至所有任務，而是「優化特定目標任務的性能，而非所有任務」。源任務（source tasks）的存在，純粹是為了作為達成最終目標的墊腳石。**

## Related Notes

- Training Trick

### **1.2. 監督式學習中的啟示**

課程學習的概念並非強化學習所獨有。在監督式學習（supervised learning）領域，此概念早已被探討，並為我們提供了深刻的啟示。Bengio 等人（2009）的開創性工作為該領域奠定了基礎，他們「假設課程同時扮演著一種連續性方法（continuation method）和正規化器（regularizer）的角色」。簡單來說，從簡單樣本開始學習，相當於在一個更平滑的目標函數上進行優化，有助於引導學習過程避開不良的局部最優點，這與強化學習中透過設置簡單任務（如初始狀態靠近目標）來簡化價值函數（value function）的學習過程有異曲同工之妙。

### **1.3. 在人類教育領域的應用**

課程學習的原則在人類教育中無處不在，尤其是在智慧導學系統（Intelligent Tutoring Systems, ITS）的設計中 。在這一領域，教學過程本身常被建模為一個強化學習問題。系統（老師）的目標是選擇一系列教學活動（動作），以最大化學生的知識掌握程度（獎勵）。這與我們為人工代理設計課程的理念高度一致：兩者都旨在找到一個最優的教學序列，以適應學習者當前的知識水平或能力。

### **第二部分：開放性問題與未來研究方向 (第六章內容解析)**

在全面回顧了現有研究後，我們在第六章系統性地識別了該領域中亟待解決的挑戰。我們相信，這些問題將是推動課程學習從理論走向更廣泛應用的關鍵。

### **2.1. 自動化與整合**

目前，課程學習的流程在很大程度上仍依賴人工介入。

- **全自動化任務創建**：現有工作大多假設任務池是預先手動設定的，或透過一套半自動規則生成 。然而，這些規則的超參數往往也需要人工調校。「減少這些方法所需的人工輸入量，仍然是未來工作的一個重要領域」。真正的全自動化任務生成，要求系統能自主地探索和創造出既具備相關性又有適當難度的中間任務。
- **任務生成與排序的結合**：在我們的框架中，任務生成與排序是兩個獨立的步驟。一個更具雄心的方向是將兩者結合，直接生成課程中的「下一個」任務 。這要求一個生成器不僅能創造任務，還能理解學習者當前的狀態，並為其量身打造下一步的學習內容，這是一個極具挑戰性但潛力巨大的方向。

### **2.2. 知識的表示與遷移**

- **知識遷移類型的靈活性**：目前的研究在課程中遷移的知識類型通常是固定的，例如始終遷移價值函數或策略 。然而，不同任務或學習的不同階段，可能受益於不同類型的知識。我們因此提出一個開放性問題：「除了決定從哪個任務進行遷移，我們還可以問，應該從該任務中提取和遷移什麼」。未來的系統或許能夠動態地決定是遷移一個高層次的技能（option）、一個環境模型，還是一個底層的策略。
- **課程的重用與真實世界遷移**：設計課程的計算成本可能非常高昂，有時甚至超過直接學習目標任務 。為了攤銷這一成本，我們必須考慮課程的重用性。正如我們所指出的，「一種攤銷成本的方法是學習一個課程來訓練多個不同的代理，或解決多個不同的目標任務」。另一個極具價值的方向是「模擬到真實」（sim-to-real）的課程學習，即在模擬環境中生成課程，然後將該課程（而非學到的具體策略）應用於訓練物理機器人 。

### **2.3. 基礎理論的完善**

- **理論結果的匱乏**：課程學習的有效性在大量實驗中得到證明，但我們仍然缺乏堅實的理論基礎。我們坦言，「儘管經驗證據顯示課程是有益的，但關於它們為何以及何時有用，以及應如何創建它們的理論結果卻很缺乏」。建立此類理論，例如為任務難度定義一個如監督式學習中那樣的量化指標，將是該領域走向成熟的關鍵一步。
- **對人類設計原則的理解**：人類，即使是非專家，也往往能直觀地設計出有效的學習課程 。深入研究人類設計課程時所遵循的內在原則，將為自動化演算法提供寶貴的啟示。我們相信，「如果我們能找到非專家在設計和／或排序更有‘趣味性’的中間任務到課程中所遵循的一些通用原則，我們就可以將這些見解融入到為任何任務領域生成有用源任務的自動化過程中」。

---
layout: page
title: "The Flexible Job Shop Scheduling Problem: A Review"
description: "Schedule classes, problem variants, and solution families, connected back to RL."
category: research paper
order: 15
---

> Dauzère-Pérès, Stéphane, et al. "The flexible job shop scheduling problem: A review." European Journal of Operational Research 314.2 (2024): 409-432.


## Introduction

- Job shop 跟 Flexible job shop 的差別在於，Flexible job shop 需要多決定這個 job 應該要放在哪一個 machine 上做 (assignment)，而不是只決定 job 在 machine 上的 sequence
   - 難度增加
      JSP領域最成功的精確演算法，例如Carlier & Pinson（1989）提出的分支定界法，其效率高度依賴於「高效解決單機排程問題」的能力 。該方法的核心思想是將整個JSP問題分解，通過解決各個單機排程子問題來計算出強而有力的下界（lower bounds），從而有效修剪搜索樹。然而，在FJSP中，由於工序的機器歸屬在決策前是未知的，我們無法預先確定任何一台機器的工序集合，自然也無法將問題分解為獨立的單機子問題。論文中明確指出：「指派決策...阻止了通過解決單機排程問題來為FJSP推導出有效的下界。因此，JSP最有效的分支定界法無法擴展到解決FJSP」 。這意味著，從JSP到FJSP的演進，不僅是困難度的量變，更是求解範式的質變。
        

<img src="{{ '/assets/notes/fjsp-review/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

- Semi-active schedule: 所有的 operation 都已經沒有辦法在改變順序的狀況下往前推
- Active schedule: 不論如何兩兩交換 operation 的順序 (同機台上)，都無法再不犧牲一個的完工時間下讓另一個 operation 提前開始
- Non delay schedule: 只要機器空了就直接 assign operation 上去，不允許 forced idle time

> **Note**
> optimal solution 存在於 active schedule 裡面不是在 non-delay 上，就代表**最佳解並非一定是 non-delay 但是必須是 active**

**接下來中間的章節在講所有可能的或是重要的 $$\alpha$$, $$\beta$$, $$\gamma$$** 

下面我覺得就是比較酷的我有筆記一下

## Additional characteristics

> Our objective in this section was to systematically categorize and explore the additional complexities,or characteristics,that researchers have incorporated to bridge the gap between the theoretical model and industrial practice

### **Deconstructing Job Routes Beyond Simple Sequences**

過去的排程都是在解 Flexible Job Shop 並且都假設每個 job 都是有一個 pre-defined 的路線 (operation precedence)，但是其實這個被過度簡化了，接下來會介紹應該需要考慮什麼

Assembly and Disassembly ( Sequencing vs. Parallelism)

- 一個 operation 可以有不只一個 predecessors 或是 successor
- 如下圖，$$O_1$$ 就會被分成 $$O_2$$ ,$$O_3$$ ，等前面兩個做完才能開始 $$O_4$$
- 這個問題又可以被分成中間的這個兩個 operation 能平行 (e.g. 拆解組裝工程) or 必須決定一個順序 (e.g. 同個部件需要不同的加工，才能到下游)

>[!note]
>Retains the core assignment and sequencing decisions of the FJSP but applies them to a more complex job structure.

<img src="{{ '/assets/notes/fjsp-review/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

### **The Reality of Multiple Resource Requirements**

過去的假設是每個 operation 只需要一個機台，若是多個 operation 都需要同樣的輔助工具時就不能有這個假設。所以通常會抽象化這件事情把 machine 變成 resource，這種 resource set 裡會有各種 combination of operators (different skills, or tools and fixtures)。文中有介紹兩中的解法以及表示法，有興趣可以多看看

### **Scheduling in Uncertain Environments**

uncertain process time, machine failure，這些都是常見的需要 stochastic scheduling 的地方，但是 FJSP 似乎比較少考慮這塊。通常人們在考慮這塊的時候用的都是 Monte Carlo simulation 來計算 expected performance 這種方式可以拿來求解找出目前最好的排程

### **Other Noteworthy Characteristics**

- Cyclic Scheduling: 這是 long-term schedule，訂單會源源不絕的進來所以不會有結束的一天，在這個狀況下 $$C_{max}$$ 就沒有意義了，反而需要考慮的是 cycle time (long-term production rate)
- Variable Processing Times: process time 通常都作為 fixed deterministic data，但是在某些情況下像是如果考慮 aging 就會是 dependent variables；考慮的是 green manufacturing 的問題下就會是 decision variables 因為如果機台全力運轉就會早成大量能源被消耗，所以當 process time 是 d.v. 就可以藉由控制 operation 的 process time 來節能讓 machine 不要總是全力運轉

## Solution approaches

> When machine flexibility is very high, i.e., when operations can be performed on many machines, finding very good or even optimal solutions is easier. The benchmark instances of Hurink et al.(1994) discussed in Section 3 are a well-known example, where the most flexible instances HUdata/vdata are easier to solve com-pared to instances HUdata / edata and HUdata / rdata. This can be explained by the fact that the total number of solutions increases with machine flexibility, but also the number of very good solutions which are thus easier to reach.


### MetaHeuristic

Deriving important structural properties to guide the search process in relevant and promising solution area / Integrating metaheuristic components that are extended and improved specifically for the FJSP.

> **Note**
> 其實 meta-huristic 過去引導學習的方法感覺更適合拿來引用在 RL，因為這兩個方法都有種找尋解答的過程。說不定一些評量方法或是機制能夠用來 guide RL

**Trajectory-based meta-heuristics**

> Algorithms working on single solutions, mainly local search approaches, form a search trajectory. Trajectory-based meta-heuristics are widely applied to solve FJSPs, including Simulated Annealing (SA), Tabu Search (TS) and Variable Neighbourhood Search(VNS).

- Tabu Search (TS): Explores the neighborhood but keeps a "tabu list" of recent moves that are temporarily forbidden. This prevents the search from getting stuck in cycles and encourages it to explore new regions of the solution space.
- Simulated Annealing (SA): Similar to local search, but to escape local optima, it sometimes accepts moves that make the solution worse. The probability of accepting a bad move decreases over time, just like a cooling metal "anneals" into a stable state.
- Variable Neighborhood Search (VNS): Instead of using just one type of move (e.g., insertion), VNS systematically switches between different, often increasingly complex, neighborhood structures to explore the solution space more effectively.
- GRASP (Greedy Randomized Adaptive Search Procedure) & Scatter Search: These are other advanced methods that guide the trajectory in different ways to balance between finding good solutions quickly (intensification) and exploring the search space broadly (diversification).
    
   > **Note**
   > 上面這個 GRASP 和 Scatter Search 可以拿來看一下是否有用，這些有切重 RL 的精神
    
> Different from trajectory-based meta-heuristics, population-based metaheuristics deal with a set of individual solutions.


這邊的內容就是比較熟悉的 Genetic Algorithm (GA), Particle Swarm Optimization (PSO), Ant colony optimization (ACO), Estimation of Distribution Algorithms (EDA) (這個我不太熟，但聽說是 GA 的進化版，建立一個機率模型 probabilistic model 來取代傳統基因演算法中的 crossover 和 mutation 操作，因為這些操作太粗暴了使得學習效率低落)

<img src="{{ '/assets/notes/fjsp-review/Screenshot_2025-09-24_at_3.48.26_PM.png' | relative_url }}" alt="Screenshot_2025-09-24_at_3.48.26_PM" loading="lazy" style="max-width: 100%; height: auto;" />

## Conclusion (Summary by Gemini)

好的，這是有關「彈性工廠排程問題 (FJSP) 的前瞻性研究議程」的要點整理：

### **FJSP (彈性工廠排程問題) 的前瞻性研究議程要點**

這個研究議程主要分為三大支柱，旨在引導 FJSP 領域走向更具實用性、理論深度與通用性的未來。

### **一、 擴展問題模型的真實性 (Expanding the Realism of the Problem Model)**

目標是讓學術模型更貼近複雜的真實工業環境。

- **非正規準則 (Non-regular criteria):**
   - 研究超越「完工時間最小化 (makespan)」的目標，例如最小化提早/延遲成本、存貨成本等。
   - 探討更符合現實的假設，例如當機器可用時，工件不能等待。
- **隨機與動態性質 (Stochastic and dynamic nature):**
   - 將研究從靜態、確定的模型，擴展到能處理隨機與動態事件的模型。
   - 考量現實世界中的不確定性，如機器故障、處理時間變動、新訂單到達等。
- **機器退化 (Machine degradation):**
   - 將機器的「健康狀況」或老化效應納入模型，這會導致加工時間變成一個依賴排程決策的變數。
   - 需將預防性維護活動與生產排程整合。
- **多重資源限制 (Multiple resources):**
   - 除了機器，還需考慮其他必要資源，如刀具、夾具、運輸系統（如無人搬運車）等。
   - 研究多個資源需同步或協調的排程問題。

### **二、 推進求解方法論的發展 (Advancing the Methodological Frontier)**

目標是精進解決這些複雜問題的演算法與工具。

- **演算法的實用性與效率 (Applicability and efficiency of algorithms):**
   - 追求設計「精簡且高效 (streamlined and efficient)」的演算法，避免不必要的複雜性。
   - 強調演算法的計算時間必須夠短，才能應用於需要高頻率決策的實際營運場景。
- **深入分析與基準案例 (In-depth analysis and benchmarks):**
   - 不僅是開發演算法，更要深入理解演算法之所以有效的「重要特性」。
   - 建立更多樣化、更能鑑別演算法性能的基準測試案例 (benchmark instances)，以促進該領域的公平比較與發展。

### **三、 範式轉移：邁向通用的解決方案框架 (A Paradigm Shift: Towards Generic Solution Frameworks)**

目標是改變目前學術界主流的研究策略。

- **提倡「由上而下」(Top-down) 的研究策略:**
   - 目前主流是「由下而上」，即為特定問題開發高度客製化的演算法。
   - 未來應投入更多精力發展「由上而下」的通用型求解器，使其能處理涵蓋多種複雜約束的通用 FJSP 問題。
- **建立通用求解器作為新標準:**
   - 一個強大的通用求解器，可以作為評估新型、專用演算法的效能基準。
   - 讓業界在面對新的排程問題時，能以一個現成的通用框架為基礎進行調整，而非每次都從零開始。

## Related Notes

- Combinatorial Optimization
- CP vs MIP

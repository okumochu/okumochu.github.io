---
layout: page
title: "Ch 2 Deterministic Models Preliminaries"
category: lecture
course: "Production and Operations Scheduling"
order: 2
---

## Notation  $$\alpha\vert \beta\vert \gamma$$

> $$\alpha$$ **生產環境 ,** $$\beta$$ **生產限制, $$\gamma$$ 生產目標**


**生產環境**

- Single machine $$1$$
- Parallel machine $$P_m$$
   - 每個 job 會經過一個 machine ，有 m 台 machine 可以選 (single machine）
   - 如果此時在 m 台 machine 上有不同的速率時（看機器就知道做多快) = $$Q_m$$
   - 如果每個job在每個m台machine有不同速率時（看機器和哪個工作才知道做多快) = $$R_m$$
- Flow shop $$F_m$$
   - **Flow Shop** 是指 job set 裡面的job都會有相同的加工 (operation) 順序。可能從 m1~m4 ，必須通過所有加工流程
- Flexible flow shop $$FF_C$$
   - **Flexible** 是指每個加工步驟可以有不只一種機器來做(有點像下拉式選單)。 parallel + flow
- Job shop $$J_m$$
   - **Job Shop** 是 job set 裡面可以存在不同 route (加工順序)的 job。有些 job 可能從 m1~m4 。有些從 m4~m1 ，**不用通過所有機台**
- Flexible job shop $$FJ_c$$
- Open Shop $$O_m$$
   - **Open Shop**是指每個 job，本身就可以有很多 route。也就是每個 job 說可以有多種加工方式

**生產限制**

- Release Date $$r_j$$
- Sequence dependent setup time $$s_{k,j}$$
- Preemption $$prmp$$
- Precedence constraint $$prec$$
   - 指某些工作必須先等某工作完成才能做 (不代表要相鄰著做)。如果至多一個 predecessor 和 successor 就會是chain。如果至多一個 predecessor 就是 outtree，如果至多一個 successor 就是 intree。for single or parallel
- Batch processing
   - job會被一群一群的加工。b = 1就跟沒有batch processing一樣
- Breakdown $$brkdwn$$ (machine availability constraint)
   - 某些machine可能有時不能使用
- Machine eligibility constraint $$M_j$$
   - 不是所有 machine 都可以加工所有 job，此符號用來表示可以做該 job 的 machine。使用於 $$P_m$$ 因為需要對於 job machine 的選擇上有限制。
- Permutation $$prmu$$
   - 適用於 $$F_m$$，每個machine上的加工的job順序都必須相同
- Blocking $$block$$
   - 加工和加工之間可以有多少buffer存放。如果 block = 0，代表下游要有人接否則上面那個completed job不會release出來。for flow shop
- No-wait $$nwt$$
   - 當上一個加工做完時，下一個要馬上開始做，如果沒辦法達成的話，就要等到可以達成才做。for flow shop
- Recirculation $$rcrc$$
   - 循環加工，operation會需要重複做。job shop 會出現 (半導體）
- Job Families $$fmls$$
   - 一個 job family 會有很多 job，每個 job 在之間一個接著一個做都不需要有 set up time。只有從一個 job family 換成下一個 job family 才需要 set up time

**目標函數**

- $$C_{i,j}$$, $$C_{max}$$
- $$L_j$$,  $$L_{max}$$, $$T_j$$, $$U_j$$
- Flowtime $$C_j - r_j$$
- Total weighted
   - completion time
   - tardiness
   - number of tardy job
- Make span, Maximum lateness
    
   <img src="{{ '/assets/notes/pos-ch2-deterministic-models-preliminaries/Screenshot_2025-03-12_at_5.10.53_PM.png' | relative_url }}" alt="Screenshot_2025-03-12_at_5.10.53_PM" loading="lazy" style="max-width: 100%; height: auto;" />
    

**Dominant Set** 

> $$C_j$$ completion time for each job 
$$Z_S = f(C_1, C_2,...,C_n)$$ objective value of schedule $$S$$ (e.g. flow time, makespan)
存在一個 dominant set $$D$$，並且有一個 arbitrary 排程 $$S$$ 並且不屬於 $$D$$
這個 $$S$$ 的每個 $$C_j$$ 將都會大於等於 $$D$$ 裡面的排程 $$S^\prime$$ 裡每個 $$C^\prime_j$$，使得 $$S$$ 的排程的目標值將會最多跟 $$D$$ 的排程一樣好而已 $$Z_S \le Z_{S^\prime}$$


所以要找最佳解的話，從這個 $$D$$ 找就足夠了

**Regular Measure**

> 指如果你的任意一個 job 的完工時間變長了，objective 就會變差。就是一個 Regular Measure。 e.g. $$\sum T_j$$

- $$\sum T_j + \sum E_j$$ 就不是一個regular measure

**Classes of Schedules**

- **Non-Delay**: 有 job 能做就得做，不能 idle。除非不得已，所有machine都在忙
   - **A schedule anomaly**
        
      > 代表排程已經是non delay，沒有force idle，結果比適當的 force idle 還要差

        
      <img src="{{ '/assets/notes/pos-ch2-deterministic-models-preliminaries/Screenshot_2025-04-06_at_4.19.05_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_4.19.05_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
- **Active**: 不存在任何兩job在交換順序後，其中一個job完成更早，另一個job沒有完成的更晚 (可以持平)
- **Semi-active**: 不存在任何job在不交換順序情況下，沒有任何job可以提早做完

<img src="{{ '/assets/notes/pos-ch2-deterministic-models-preliminaries/Screenshot_2025-03-12_at_3.34.58_PM.png' | relative_url }}" alt="Screenshot_2025-03-12_at_3.34.58_PM" loading="lazy" style="max-width: 100%; height: auto;" />

### Complexity

Effort for scheduling problem (in generalization sense)

- Merge Sort $$O(nlogn)$$ e.g. SPT rule
   - 切開不是問題，花時間的是在 merge
   - 樹大約是 $$log(n)$$ 的高度，每層要做 $$n$$ 次的比較
   - Example
        
      <img src="{{ '/assets/notes/pos-ch2-deterministic-models-preliminaries/Screenshot_2025-04-06_at_4.40.17_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_4.40.17_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
      <img src="{{ '/assets/notes/pos-ch2-deterministic-models-preliminaries/Screenshot_2025-04-06_at_4.41.29_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_4.41.29_PM" loading="lazy" style="max-width: 100%; height: auto;" />
        
- Johnson's rule:
   - $$F2\vert  \vert C_{max}$$。上游越短越先，下游越短越後面

**NP-Hardness**

- **P:** 可以在**多項式時間內解決**的問題 (效率高，電腦能在合理時間內解出來)
   - 你有一張地圖，能很快走出去
- **NP:** 可以在**多項式時間內驗證答案**的問題 (不一定能快解，但給你一個答案你可以很快確認對不對)
   - 你不知道怎麼走，但有人說他知道一條路，你拿到路線後一驗證就知道對不對
- **NP-hard:** 是**至少跟所有NP問題一樣難**，甚至**可能更難**的問題。
   - 迷宮不只大，而且可能根本沒地圖，也沒有簡單的驗證方法；你只能一直亂試
   - 不是所有 NP-Hard 問題都一樣難
      - 有些 NP-hard in ordinary sense 可以用 pseudo-polynomial algorithm 來解 (e.g. $$1\vert \vert \sum{T_j}$$)
      - 更難的問題叫做 strongly NP-hard (e.g. $$1\vert r_j\vert \sum{C_{ma} }$$)
   - Example
        
      <img src="{{ '/assets/notes/pos-ch2-deterministic-models-preliminaries/Screenshot_2025-04-06_at_5.07.03_PM.png' | relative_url }}" alt="Screenshot_2025-04-06_at_5.07.03_PM" loading="lazy" style="max-width: 100%; height: auto;" />

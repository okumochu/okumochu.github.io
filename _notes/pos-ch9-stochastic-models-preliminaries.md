---
layout: page
title: "Ch 9 Stochastic Models Preliminaries"
category: lecture
course: "Production and Operations Scheduling"
order: 9
---

## Distributions and Classes of Distributions

先對一些機率分佈做一些介紹 (連續和間斷)，以及其**機率**以及**尾機率** ( $$1-P(x)$$ )基於數學上的特性在排程上的分類。通常這些分佈都是對於環境的假設，基於這些架設就可以對某些排程方法做一些 performance 上的證明

### Classic Distributions

- Exponential
- Geometric
- Determinisitc

### Classes of Distribution

接下來就要了解這些分佈**在數學上的特性上應用在排程**會怎麼分類

- IFR (Increasing Failure Rate)
- IMRL/DMRL (Increasing/Decreasing Mean Residual Life)
- NBUE/NWUE (New Better/Worse than Used in Expectation)

## Stochastic Dominance

比較兩個隨機變數誰比較大，以下有多種 stochastic dominance 的概念

- Stochastic Dominance Based on Expectation:
- 分佈優越:
- 更高階的 stochastic dominance:

## Impact of Randomness on Fixed Schedules

本節討論當排程順序固定時，處理時間等參數的隨機性如何影響目標函數的期望值

- Expected Objective Value
- Varience

> **Note**
> 此節說明若在不確定性高的環境，盡量採取 robust scheduling。過於精細的排程會被隨機性打敗

## Classes of Policies

本節定義了隨機排程中決策策略的幾種主要類別，依據決策者可利用資訊的程度和是否允許插斷來區分

- **非中斷靜態清單策略（Nonpreemptive Static List Policy）**：在時間零即對所有作業排定一個**固定優先順序清單**（Permutation），整個過程中該清單順序不變。每當任一機台空閒時，清單中尚未加工的最前面的作業被選來加工。此類策略又稱「排列策略」，類似確定性模型中的固定優先序規則。適用情境通常是**所有作業在零時刻皆已釋放**且不允許中途插入新作業。
    
   *例：* 考慮單機排程，有3個作業同時可用，各自處理時間和交期皆有兩點分佈（處理時間2或8，交期1或5，各以0.5機率）。採用一固定順序排程時，可計算**按時完成作業數**的期望值：例如若清單順序為作業1→2→3，則第一個作業按時完成概率0.25（需處理時間短且交期長），第二個作業按時完成概率0.125，第三個作業必遲交，期望按時完工數=0.375。此順序下預期遲交作業數2.625。
    
- **可中斷靜態清單策略（Preemptive Static List Policy）**：同樣在時間零預先給出所有作業（包括未釋放者）的優先順序清單，但允許作業釋放後打斷正在執行的較低優先順序作業。當新作業釋放且其在靜態清單中的位置高於當前加工作業，則立即**中斷**目前作業，改先處理新作業。此策略適用有不同釋放時間的情況，但仍屬靜態因優先序預先固定不變。
- **非中斷動態策略（Nonpreemptive Dynamic Policy）**：每當機台空閒時，允許決策者根據**當時可用的最新資訊**來選擇下一個加工作業。資訊可包含當前時間、各機台上正在加工及等待的作業狀態等。選定作業後需執行至完成（不可中斷）。由於可滾動調整順序，這類**動態規則**通常能比靜態清單達到更佳績效。
    
   *例：* 延續上述單機3作業範例，在非中斷動態策略下，每次作業完成後可根據剩餘作業的狀態重新決策。結果顯示動態策略可略提高按時完成的期望數，例如可達0.4375（遲交數2.5625）。動態決策利用了即時資訊（如若第一個作業結束時發現其餘作業交期已過則不再加工等），因此表現優於任何固定順序。
    
- **可中斷動態策略（Preemptive Dynamic Policy）**：決策者在任何時間點都可根據當前所有資訊調度作業，並可**隨時中斷**加工中作業以切換更高優先度的作業。這是最靈活的策略類別，理論上包括所有可能的排程決策序列（也是分析最困難的一類）。
    
   *例：* 在單機範例中，可中斷動態策略允許隨時插入高優先作業，進一步略微提高按時完成率（如第一作業依然0.25，整體遲交數可能稍減）。不過該範例中，由於作業同時到達且處理時間短，非中斷與可中斷動態的差異不大。一般而言，可中斷動態策略在具長 processing time 或頻繁緊急插單情境下優勢明顯。
    
   | **Policy Category** | **Priority Decision Timing** | **Allows Preemption** | **Key Features** |
   | --- | --- | --- | --- |
   | Nonpreemptive Static List Policy | Predetermined at time zero | No | Fixed priority order list, suitable for scenarios where all jobs are released simultaneously |
   | Preemptive Static List Policy | Predetermined at time zero | Yes | Allows newly released high-priority jobs to interrupt current jobs |
   | Nonpreemptive Dynamic Policy | Based on latest information when machine is idle | No | Can adjust order on a rolling basis, usually achieves better performance than static lists |
   | Preemptive Dynamic Policy | At any point in time | Yes | Most flexible policy category, can interrupt at any time to switch to higher priority jobs |

> **Note**
> 總體而言，策略的選擇取決於對隨機資訊的允許使用程度：**靜態策略**僅用初始資訊，**動態策略**充分利用過程中揭露的新資訊；**非中斷**著眼避免切換成本，**可中斷**則追求彈性和及時響應。理論分析常先在限制較多的策略類別中尋找最優解，再討論該解在更廣策略類別中的最優性

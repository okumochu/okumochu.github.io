---
layout: page
title: "A Survey on Curriculum Learning"
description: "Predefined vs automatic curricula and self-paced learning, summarized."
category: research paper
order: 9
---

> Wang, X., Chen, Y., & Zhu, W. (2021). A survey on curriculum learning. *IEEE transactions on pattern analysis and machine intelligence*, *44*(9), 4555-4576.


# Curriculum Learning in a nutshell

Curriculum learning (CL) is a training strategy that **trains a machine learning model from easier data to harder data**, which imitates the meaningful learning order in human curricula. As an easy-to-use plug-in, the CL strategy has demonstrated its power in **improving the generalization capacity and convergence rate of various models** in a wide range of scenarios such as computer vision
and natural language processing etc.

# Introduction

最早在做 Curriculum Learning 的是在做影像分類，會先從比較簡單的圖片的 batch 開始學習，剛剛是講的是 CV 領域而且是把它當作一個訓練的 trick 而不是一個方法。後面慢慢有人在研究這個部分才有應用到 Natural Language Processing (NLP)、Reinforcement Learning、Graph Learning 等問題，並且主要都是以 **improving the model performance on target tasks and accelerating the training process** 為目的 (並且非常好實作 plug-and-play)

<img src="{{ '/assets/notes/curriculum-learning-survey/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

# Definition

Orginal Three Conditions:

1. 資訊量 (entropy) 要逐漸提升 ( i.e. 越後面的課程有越高的機率抽到越難的 example
2. 越難的 example 要有越多的資料
3. 最後一個課程 (= final task) 會包含前面的所有課程且抽取倒的機率相同 (uniform)

這些是最早的設定，當然除了簡單到難其實還有很多其他維度: 

1. “from easy & diverse to hard” to avoid overfitting
2. “from easy & imbalanced to hard & balanced” data to alleviate the
severe class imbalance in human attribute analysis

到後面的發展就會讓這個難度可以被動態調整，使 model 能夠動態決定要學什麼難度的課程 (打破了原本的條件)，有趣的是某些研究發現這種像是 model free 的做法的排出來來的課程排序竟然是從難學到簡單，這個就是另一個 research topic 叫做 hard example mining (HEM) 

最後也不把課程限制在 data level (打破更多條件)，前面都是建立在 learning dataset 上調整簡單到難，但是課程的設計也可以是著重在調整 loss function、drop out rate、search space

# ANALYSIS ON EFFECTIVENESS OF CL AND SUITABLE APPLICATION SCENES

這個區塊在探討課程學習這個 trick 之所以有用背後的意涵是什麼? 會以 "最佳化問題” 以及 “資料分佈” 的角度來說明為何課程學習能給我們更好的泛化能力以及收斂速度。基本上可以說 CL 在做的事情就是 guide and denoice

## Theoretical Analysis on CL

**from the prespective of “optimization problem”: continuation method**

這是一個最佳化的方法主要處理的是 non-convex critiria，這個方法 idea 主要如下圖

1. 會先考慮一個相對平滑的 search space (簡單) 的問題，先讓模型知道一個大致上的 global view
2. 然後逐漸把問題變難，類似模擬退火的概念。透過這樣的概念可以使其更加接近 global optima

> Weinshall, D., Cohen, G., & Amir, D. (2018, July). Curriculum learning by transfer learning: Theory and experiments with deep networks. In International conference on machine learning (pp. 5238-5246). PMLR.


<img src="{{ '/assets/notes/curriculum-learning-survey/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

**from the perspective of “data distribution”**

在 large-scale deep learning 的任務上，會包含很多各式各樣的資料。這些資料有可能會是錯誤標註或是充滿 noise 的資料，並且有這些屬性的資料通常都是比較困難的。所以用 Curriculum Learning 也可以直覺上來說先避開那些資料，並且先有一個 high density 的定錨，最後再用其他比較 noisy 的 data 做修正

> Gong, T., Zhao, Q., Meng, D., & Xu, Z. (2016). Why curriculum learning & self-paced learning work in big/noisy data: A theoretical perspective. Big Data and Information Analytics, 1(1), 111-127.


<img src="{{ '/assets/notes/curriculum-learning-survey/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

左圖是表示 training data 以及 target data，train data 有厚尾是因為裡面有很多 noisy data。至於右邊是指 CL 的過程從最簡單的 data point 開始學 (就會先學到那個 peak)，慢慢變成紅色的 uniform 代表所有 data 最終會平等的被抽到 (在學比較 noisy 的那幾個)

## Suitable Application Scenes of CL

根據上面的討論，可以把 CL 的作用分為兩個 **“to guide” and “to denoise”**  e.g. 如果有 imbalance data 就可以從 balance data 訓練到 imbalance data

<img src="{{ '/assets/notes/curriculum-learning-survey/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

# CL DESIGN: A GENERAL FRAMEWORK

從前一章節可以知道為什麼大家愛用 CL，這個區塊來介紹要如何設計 CL

<img src="{{ '/assets/notes/curriculum-learning-survey/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

## The General Framework of Difficulty Measurer + Training Scheduler

這個架構下就是把所有的資料先使用 difficulty measurer 做排序然後按照順序訓練，至於訓練的步數就是由 training scheduler 決定

## **Predefined CL**

both the Difficulty Measurer and Training Scheduler are designed by human prior knowledge with no data-driven algorithms involved

- Common Types of Predefined Difficulty Measurer
   - Complexity: 結構上的複雜度 e.g. 語法難度
   - Diversity: 資料的分佈有多不一樣，越難代表有很多不同分佈的資料
      - 可以用 information entropy 衡量
      - 如果是 tabular data 就可以用 Mahalanobis distance
   - Noise: 通常是影像任務就會有雜訊比可以來衡量
        
      <img src="{{ '/assets/notes/curriculum-learning-survey/image-5.png' | relative_url }}" alt="image 5" loading="lazy" style="max-width: 100%; height: auto;" />
        
- Common Types of Predefined Training Scheduler
   - 這邊介紹的都比較 task agnostic，就是一個每過多少個 update step 就換下一個課程
      - 這邊指的下個課程有兩種，一種是有包含上一個課程 (e.g. 包含上個課程的訓練資料)，一種沒有 (sequencial task training)。可想而知沒有包含的那個可能會有災難性遺忘
        
      <img src="{{ '/assets/notes/curriculum-learning-survey/image-6.png' | relative_url }}" alt="image 6" loading="lazy" style="max-width: 100%; height: auto;" />
        

> **Note**
> 雖然 predefined CL 時做起來簡單直觀，但是問題是一個 difficulty measurer and training scheduler 不好選也不好訓練，通常就是不停的試錯。就是你真的很強精通 domain 知識參數也都調超好，這種類型的方法要可行背後會有一個強假設就是 **domain 覺得的難跟模型覺得的難要相同**

## **Automatic CL**

both the Difficulty Measurer and Training Scheduler are learned by data-driven models or algorithms

這樣的自動課程學習被大量應用在 Deep RL task

<img src="{{ '/assets/notes/curriculum-learning-survey/image-7.png' | relative_url }}" alt="image 7" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/curriculum-learning-survey/image-8.png' | relative_url }}" alt="image 8" loading="lazy" style="max-width: 100%; height: auto;" />

### **Self-Paced Learning (SPL)**

這個方法模擬學生自主學習的時候知道自己掌握難易度，所以 student 會用自己的 loss 來決定自己應該要在什麼時候學進階的課程。並且這是一個主要的 CL 分支，並且主要是在機器學習上做應用

- The Orginal Version of SPL
   - 假設有 N 筆資料，SPL 在機器學習裡面除了 loss function 前面會多一項每筆資料的權重 $$v$$ 以及一個約束 $$v$$ 的 regularization term $$g$$
      - 在最原始的 SPL 的 $$g(\boldsymbol{v}; \lambda) = -\lambda \sum_{i=1}^{N} v_i.$$
        
      $$ \min_{\boldsymbol{w};\boldsymbol{v}\in[0,1]^N} \mathbb{E}(\boldsymbol{w}, \boldsymbol{v}; \lambda) \sum_{i=1}^{N} v_i l_i + g(\boldsymbol{v}; \lambda). $$
        
   - 接下來有這個 form 之後會用 *Alternative Optimization Strategy (AOS)*
      - 這個模式就是固定 $$w$$ (parameter of model) 求出 optimal $$v$$ (weight of loss for each data point)，然後求出 optimal $$v$$ 之後反過來固定 $$v$$ 求出 optimal $$w$$
      - 然後因為 $$w$$ 改變了所以就可以重複繼續求出 optimal $$v$$
      - 然後隨著 update step 越來越多就會把 $$\lambda$$ 慢慢增大使其能夠加入更難的例子
         - 前面這項代表每個 data 帶來的 loss，如果 $$\lambda$$ 變大會使得後面這項變重要，所以就不會這麼多 $$v_i$$ 被壓成 0。反之，如果 $$\lambda$$ 很小很有可能 $$v$$ 全部都是 0，因為這樣就不會有 loss 了前面那項直接 = 0
            - 如果是求解 model parameter 就可以用梯度下降 (沒有辦法求出最佳解
                    
               $$ w^* = \arg \min_{\boldsymbol{w} } \sum_{i=1}^{N} v_i^* l_i. $$
                    
            - 如果是 Loss weight 話如果 $$g$$ = l1 norm 的話，下面這個式子就會是 convex function 用偏微分是可以求出全域最佳
                    
               $$ v_i^* = \arg \min_{v_i \in [0,1]} v_i l_i + g(v_i; \lambda), \quad i=1, 2, \dots, n.\\ $$
                    
               - 下面是 closed form solution，直觀上很好解釋就是如果這個筆資料的 Loss  $$l_i < \lambda$$ 代表這是 easy exmaple，所以需要被學，否則就是太難這階段還不適合學
            
         $$ v_i^* = \begin{cases} 1, & l_i < \lambda \\ 0, & \text{otherwise}. \end{cases} $$
            
   - 這篇文章後面有證明 SPL 的好性質 convergence, robustness，有興趣可以自己去看
- Soft SP-regularizers
   - 這個超參數 $$\lambda$$ 以及其 regularize term $$g(v;\lambda)$$是最重要的，只是 oringal version 的 $$v$$ 沒有彈性只有 1 或 0 (間單或難)，但是其實應該中間是個光譜所以接下來就是需要改進 $$g$$
      - 以下是一個 table 的 regulizer and closed form solution
            
            
         <img src="{{ '/assets/notes/curriculum-learning-survey/image-9.png' | relative_url }}" alt="image 9" loading="lazy" style="max-width: 100%; height: auto;" />
            
         <img src="{{ '/assets/notes/curriculum-learning-survey/image-10.png' | relative_url }}" alt="image 10" loading="lazy" style="max-width: 100%; height: auto;" />
            
- Prior-embedded SPL
   - 在固定 regulizer 時，有時候想要給一些 domain prior e.g. 某些資料非常重要不管怎樣都要學
   - 或是 diversity prior: Important examples should be scattered across the data range to help learn global data knowledge
      - 這個部分就會加上一些 LASSO 的想法 (b = sum of group)，使其能夠有 $$v$$ across group
         - 因為如果是正的 l2 norm 會使其把值壓小，但是負的會鼓勵有值
        
      $$ g(\boldsymbol{v}; \lambda, \gamma) = -\lambda \sum_{i=1}^{N} v_i - \gamma \sum_{j=1}^{b} |\boldsymbol{v}^{(j)}|_2, $$
        
      > **Note**
      > 本研究後面也有提到更多的想加 prior 進去以及如何實際加進去的方法，有興趣可以自己看。並且後面也有提到怎麼調整 $$\lambda$$ (最值觀的就是逐漸退火)
        
- Applications of SPL
   - 用在很多 CV task (e.g. segmentation, image classification, object detection, etc)，以及其他的傳統的 ML task (e.g. clustering)

### Transfer Teacher

有這種方法的 intuition 是因為如果是 self-learning 的狀況下前期其實學生完全沒有任何知識，這個時候讓他去選下一個 task 要做甚麼是困難的

所以使用一個已經完全訓練好的 teacher，由 teacher 決定問題的難度。至於測量的方法有很多種像是 Loss, Cross entropy

<img src="{{ '/assets/notes/curriculum-learning-survey/image-11.png' | relative_url }}" alt="image 11" loading="lazy" style="max-width: 100%; height: auto;" />

### RL Teacher

前面的都是改變 difficulty measurment (問題多難) 的方法，但是 training scheduler (要訓練多久) 都是需要 predefined

所以使用強化學習，就由學生的表現來做選擇課程的這個決策 (直接把預測難度衡量內化在模型裡面)，

<img src="{{ '/assets/notes/curriculum-learning-survey/image-12.png' | relative_url }}" alt="image 12" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> 這也是最 auto CL 的方法，並且適合做 multi-task learning 

### Other Automatic CL

用了其他最佳化方法來選擇最佳課程來學習

<img src="{{ '/assets/notes/curriculum-learning-survey/image-13.png' | relative_url }}" alt="image 13" loading="lazy" style="max-width: 100%; height: auto;" />

## How to Choose A Proper CL Method

這邊會討論很多過去文獻在設計 CL 時的不同的結論，有興趣可以自己去看，滿多有趣的統整。總之如果有 prior knowledge 就盡量自己多定義一點，用 hybird CL 也可以，否則就用 auto CL

接下來就是討論這些方法的 computational cost (用 big $$O$$ 表示 i.e. 如果要每比資料都預測一次難度就會是 $$O(n)$$)

# Discussion

## Easier First versus Harder First

這邊提到的是有另一個方法叫做 *hard example mining (HEM)* 。這個方法的假設就是越難的 data 有越多的資訊量，所以要先學比較難的資料

如何選用是個難題，但是直覺上來說如果資料集比較乾淨用 HEM 應該會比較適合，因為資訊量比較不會有雜訊。反之，如果很髒且 task 實在是太難的話用 CL 才比較有機會學起來

## Relationship Between CL and Other Concepts

<img src="{{ '/assets/notes/curriculum-learning-survey/image-14.png' | relative_url }}" alt="image 14" loading="lazy" style="max-width: 100%; height: auto;" />

## Related Notes

- Training Trick

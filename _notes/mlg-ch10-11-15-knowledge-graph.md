---
layout: page
title: "Ch 10, 11, 15 Knowledge Graph"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 10
---

# Knowledge Graph Embeddings

Knowledge Graph (KG) 通常是異質圖，並且有兩個特色 

1. 圖超級大: 有很多節點跟邊 
2. 通常都是不完整的: 很多 missing edges

## Knowledge Graph Completion

這邊的任務跟傳統的 link prediction 不同

- **Task:** given (head $$h$$, relation $$r$$) predict missing tail $$t$$
   - 模型的輸入會是一個 (head $$h$$, relation $$r$$) 然後 output shallow embedding of head $$h$$
   - 並且若有 link 存在 output embedding 要跟 $$r$$ embedding 非常靠近
- **Problem properties:**
    
   為了選擇合適的模型，必須考慮關係具有哪些數學屬性。不同的模型對這些屬性的表達能力不同
    
   - **Symmetry:** 室友關係 (*h* 是 *t* 的室友 ⇒*t* 是 *h* 的室友)。
   - **Anti symmetry:** 父親關係。
   - **Inverse:** 老師與學生
   - **Composition:** 母親的母親 = 外祖母
   - **1-to-n:** 一個老師有多個學生

## Models

1. **TransE** 
   - Main design: assume $$h + r  = t$$
      - form language model: King - Man + Woman = Queen.
   - Pros and Cons
      - 這樣的設計可以 handle inverse 的關係，因為可以直接向量相減來達成
      - 不太能處理 symmetry 關係，因為 $$*h+r=t*$$ 且  $$*t+r=h$$* 使得 $$h = t$$
      - 同樣的道理 1-to-n 也會有問題因為不同的 head $$h$$ 加相同的 $$r$$ 無法得到相同的 $$t$$
         - 這樣就無法滿足多個 head 對應到同一個 tail
   - Goal: minimize the gap of $$h+r$$ and $$t$$
2. **TransR** 
   - Main design: like **TransE,** but project the $$h$$, $$t$$ to the relation $$r$$ space first
   - 這樣就可以解決上述 1-to-n 問題，不同的 $$h$$ 會先被轉換成相同的點，這樣加相同的 $$r$$ 就可以得到相同的 $$t$$
3. **DistMult** 
   - Main design: 直接計算 <head, relation, tail> cosine similarity
        
      $$ f_r(h, t) := \sum_i h_i r_i t_i $$
        
   - 適合處理 symmetry 關係，但是 inverse 關係會有問題，因為乘法有交換律
   - 後面又有出了一個 **ComplEx** 引入複數來解決這個問題

# Reasoning in Knowledge Graphs

剛剛的內容 KG completion (i.e. link $$r$$ exist between $$h$$ and $$t$$ ?) 可以想像成 1-hop reasoning (是否 tail $$t$$ 是 $$(h,r)$$ 的答案)。所以 Multi-hop reasoning 其實在做的事情就是是否這個 $$v_a$$ query $$q$$ 的答案

<img src="{{ '/assets/notes/mlg-ch10-11-15-knowledge-graph/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

(待補) answer query 這件事情在 KG 這樣 incomplete 的狀況下有什麼好方法能做。以下下面的 foundation model

# Foundation Models for Knowledge Graphs

Foundation models are general-purpose machine learning models pre-trained on massive datasets that can be used for zero-shot or few-shot learning across multiple downstream tasks

LLM 已經有很好的成果了，如何把這些技巧 adapt to KG，需要解決 KG 的結構變異度相較於文字 (文字都是同質的) 太高的問題 (不同的實體、不同的 relation)

## **The Core Challenge: Transductive vs. Inductive Learning**

To understand why foundation models for KGs are difficult to build, you have to look at how traditional models operate:

- **Transductive Link Prediction:** Traditional models, like shallow encoders or Relational GCNs, memorize the specific entities and relations they are trained on. They can predict missing links within that specific graph, but if you introduce a brand new entity, you must completely retrain the encoder.
- **Inductive Link Prediction:** A true foundation model must be able to transfer knowledge from one KG to a completely unseen KG with entirely different relation types. Traditional models fail at this because they rely on fixed relation types

## **The Solution: The InGram Architecture**

這個方法把 relation & node feature 分開訓練取得 embedding

原本的是 relation 只是幫助 node message passing，但是不同的 relation 可能會影響到 message passing，雖然異質圖目前是有辦法應對不同的 relation，但是沒有辦法處理新的 relation (i.e. 不在訓練資料裡面的 relation)。但是如果 model relation 就可以有能力外推新的 relation 會怎麼影響到 model

- **Relation Graph Construction:**
   - Instead of just looking at entities, InGram builds a secondary graph where the *relations themselves* are the nodes
   - The connections (adjacency matrix) between these relations are calculated based on how often entities share head and tail roles across different relation pairs
- **Relation-Level Message Passing:**
   - A Graph Neural Network (GNN) processes this relation graph to generate embeddings for each relation.
   - This allows the model to learn how relations interact globally, conceptually similar to how a Transformer learns the connections between all tokens in a sentence.
- **Entity-Level Message Passing:** Once the relations are embedded, InGram uses an attention mechanism to pass messages between the actual entities in the original KG
    
   <img src="{{ '/assets/notes/mlg-ch10-11-15-knowledge-graph/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />
    

(待補) 後面有更多細節就先跳過

---
layout: page
title: "Ch 14 Advanced Topics in Graph Neural Networks"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 14
---

這個章節要探討的是三個額外的內容，這些內容算是 GNN 的延伸研究

## **PRODIGY: In-Context Learning Over Graphs**

傳統上當有不同的下游任務的時候，可以多 train 幾個 model，不然就是訓練多幾個 head，這些 head 來運用 GNN 這個 feature extractor **(inspired by LLM)**

- 當要學習一個任務的時候除了用 loss function 訓練各種不同的 task，要再加上一個 prompt 去說明這個任務，讓模型知道現在在學習什麼任務
- **The Challenge & Solution:** Unlike text, which is just a 1D sequence of tokens, graphs have wildly varying topologies. How do you feed a node classification task and a graph classification task into the exact same model?
   - PRODIGY solves this by converting all tasks into a universal **Prompt Graph** format. It structures the input into a hierarchical meta-graph containing the original data graphs, label nodes, and task-level edges
   - You can think of the Prompt Graph as a universal adapter. By converting every possible task (whether predicting a link or classifying an entire molecule) into a unified link-prediction task between a data node and a label node, the GNN learns a "universal language" of graph structures. The model is pre-trained using techniques like neighbor matching to naturally recognize these patterns without needing re-training
        
      <img src="{{ '/assets/notes/mlg-ch14-advanced-topics/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        

## **Conformal GNNs for Uncertainty**

動機就是點估計不夠，需要知道使用這個 model 到底風險有多少。基本上就是把 conformal prediction 加上 GNN 來使用

## **Adversarial Robustness**

就如同 image classification 有可能會被微小 (肉眼看不出來) pixel perturbation 改變 label 相同，GNN 如何透過一些手法能增強應對這些 adversarial attack 之 robustness

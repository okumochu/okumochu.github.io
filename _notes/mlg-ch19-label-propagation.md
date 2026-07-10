---
layout: page
title: "Ch 19 Label Propagation"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 19
---

Chapter 19 focuses on semi-supervised transductive node classification,essentially, how to guess the labels of unlabeled nodes when you only have a few labeled nodes in your graph. This works because of two social phenomena: 

1. **homophily** (similar nodes tend to connect) and 
2. **influence** (connected nodes adapt to share characteristics). 

Instead of relying solely on heavy neural networks, we can pass known labels along the edges of the graph.

> **Note**
> 如果有一張大圖，裡面有些 node 沒有 label，如何根據這張大圖其他有 label 的 node 推估 missing label 是什麼?

## **Basic Label Propagation (LP)**

- **How it works:** You initialize known nodes with their true labels (e.g., 0 or 1) and unknown nodes with 0.5. The algorithm iteratively updates the unknown nodes by averaging the probabilities of their neighbors until the values stop changing (convergence).
- While mathematically simple and fast per step, basic LP has severe limitations. It completely ignores node features (like user profiles or text) and relies *only* on the graph structure. Furthermore, convergence is not mathematically guaranteed; in perfectly bipartite graphs (like a user-to-product graph), the label values can theoretically oscillate back and forth forever without settling.

## **Correct and Smooth (C&S)**

- **How it works:** This is a powerful, model-agnostic pipeline. First, you use a base predictor (which could be a simple, non-graph model like an MLP) to generate "soft labels" for all nodes.
   - **Correct step:** The model calculates its errors on the *known* training data and diffuses those errors across the graph to fix systematic biases.
   - **Smooth step:** It then diffuses the corrected predictions across the graph, enforcing the assumption that connected nodes should have similar labels.
- **Extra Insight:** The real magic of C&S lies in its diffusion matrix (*A*~=*D*−1/2*AD*−1/2). By multiplying the adjacency matrix by the inverse square root of the degree matrices, it normalizes the graph. Mathematically, this bounds the eigenvalues between [-1, 1]. This is a critical stability guarantee,it ensures that as you multiply the matrix over and over to diffuse the labels, the values smoothly decay rather than blowing up to infinity.

## **Masked Label Prediction**

- **How it works:** Inspired by language models like BERT, this method treats labels as just another node feature. During training, you randomly hide (mask) a portion of the true labels and train the network to guess them using the remaining labels and node features.
- **Extra Insight:** This transforms a standard classification task into a self-supervised learning task. By forcing the model to reconstruct missing labels from its neighbors, the network is forced to learn the deep structural context and local neighborhoods of the graph, making it much more robust against overfitting than traditional training methods.

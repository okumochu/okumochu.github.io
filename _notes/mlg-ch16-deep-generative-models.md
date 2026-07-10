---
layout: page
title: "Ch 16 Deep Generative Models for Graphs"
category: lecture
course: "Machine Learning with Graphs (Stanford, 2024)"
order: 16
---

生成圖主要的目的是要觀察原本的圖的 data distribution 然後 (根據最佳化目標) 生成屬於該圖 data distribution 的額外的新內容。實際用途如 simulating social networks, discovering new drugs, designing materials, and detecting anomalies。但是實際上來說不像是圖片或是 text 這麼有結構化的格式所以需要發揮創意

<img src="{{ '/assets/notes/mlg-ch16-deep-generative-models/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />

## **Realistic Graph Generation with GraphRNN**

這種 task 的目標就是生成的新內容越接近原本的 data distribution 越好

GraphRNN solves this by framing graph generation as an autoregressive sequence problem. It generates a graph by adding nodes and edges one at a time, using a hierarchical structure:

- **Node-level RNN:** Generates a new node and initializes the state for the edge-level RNN
- **Edge-level RNN:** Takes the new node and sequentially predicts whether it should connect to each of the previously generated nodes

<img src="{{ '/assets/notes/mlg-ch16-deep-generative-models/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

<img src="{{ '/assets/notes/mlg-ch16-deep-generative-models/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

- **Scaling Up with BFS Ordering:**
   - 當 node 越來越多的時候判斷是否要連線就需要執行越來越多次 e.g. n node → n-1 次判斷
   - 所以改成用 BFS 的方式做判斷，如果從比較前面的連接的 node 就決定不連起來了，後面也就不用判斷了
        
      <img src="{{ '/assets/notes/mlg-ch16-deep-generative-models/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />
        

## **Goal-Directed Generation with GCPN**

The second major task is **goal-directed generation**, where you don't just want a "realistic" graph, but a graph that maximizes a specific objective, like drug-likeness or chemical stability

- **Action:** The GCPN inserts nodes and uses a highly expressive GNN to predict which nodes to connect
- **Reward:** The environment acts as the black box, providing a "Step Reward" (to ensure the model is taking chemically valid actions) and a "Final Reward" (to optimize for the desired property, like high drug-likeness)
- **Training:** It is trained in a hybrid fashion. First, it uses supervised training to imitate the actions of real observed graphs (**learning the basic rules of chemistry**). Then, it uses RL policy gradients to actively optimize the rewards and discover novel structures
    
   <img src="{{ '/assets/notes/mlg-ch16-deep-generative-models/image-4.png' | relative_url }}" alt="image 4" loading="lazy" style="max-width: 100%; height: auto;" />

---
layout: page
title: "Challenges and Opportunities in Deep Reinforcement Learning with Graph Neural Networks"
description: "Review of how DRL and GNNs combine, and where the open challenges are."
category: research paper
order: 3
---

## Introduction & Motivation

### Why DRL

- **Computational Efficient:** Like the advantage diff between Metaheuristic algorithms and mathematical programming (exact methods), allowing fast generation of high-fidelity solutions that are crucial in highly dynamic environments with demand for real-time decision
   - Optimizaion Technique
        
      <img src="{{ '/assets/notes/drl-gnn-review/image.png' | relative_url }}" alt="image" loading="lazy" style="max-width: 100%; height: auto;" />
        
- **Understand environment dynamics and  based solely on interactions with the environment:** produce near-optimal actions without the need for explicit prior knowledge of the underlying system

### Why DRL $$\times$$ GNN

- DRL is rapidly being adopted in various applications that involve environments and
formulations exhibiting structural relationships (e.g. Knowledge Graph)
   - GNN architectures can jointly model both structural information and node attributes, They provide significant performance improvement for graph-related downstream tasks such as node classification, link prediction, community detection, and graph classification
- The ability to train in constant changing of topology (adding node or edges) providing more robust model performance

<img src="{{ '/assets/notes/drl-gnn-review/image-1.png' | relative_url }}" alt="image 1" loading="lazy" style="max-width: 100%; height: auto;" />

> **Note**
> **Strength:** Scalable (memory and computational), Generalization acorss envs

### Taxonomy

<img src="{{ '/assets/notes/drl-gnn-review/image-2.png' | relative_url }}" alt="image 2" loading="lazy" style="max-width: 100%; height: auto;" />

## Symbiotic DRL–GNN Frameworks

In these articles, either GNN is used to improve the formulation and performance of DRL or DRL is used to improve the applicability of GNN.

### DRL Enhancing GNN

- **Neural architecture search:** It refers to the process of automatically searching for an optimal architecture of a neural network (e.g., the number of layers and the number of nodes in
layer)
   - Use RL to search the network archotecture of GNN, and the trianed agent could be used to quickly determine the optimal parameters for various use cases (high generalizability)
- **Explaining GNN predictions:** Generating explanations for DNN predictions
   - Identifying the subgraph that is most influential in generating a prediction: use DRL agent to expand from a node and generate the subgraph of the original graph. the reward is defined by the mutual information between the original predicted label and the label made by the generated graph
- **Generating adversarial attacks for GNN:** Recent studies have shown that GNNs are vulnerable
to adversarial attacks that perturb or poison the data used for training them. The atacks could be used to design defense strategies or serving as a robustness evaluation

> **Note**
> DRL good at makeing sequential dicisions (scalability, generalization, learning from interaction)

### GNN Enhancing DRL

- **Modeling relationship among agents in MADRL (multi-agent):** In MADRL, a group of agents cooperate or compete with each other to achieve a common goal. Communication among agents offers additional information about the environment and state of other agents.
   - Uses the message-passing mechanism to coordinate the behaviors of agents (communicate).
- **Modeling relationship among tasks in MTDRL (multi-task):** Exploit commonalities between multiple tasks in order to learn policies with improved returns. One of the inherent assumptions in a majority of MTDRL works is the fixed state–action spaces. This issue has been addressed by
using GNNs that are capable of processing graphs of arbitrary size, thereby supporting MTDRL in variable state–action space environment
   - Limb features are encoded in the form of node labels and edges represent the physical
   connections between the corresponding limbs, and use policy to predict the actions for different part
- **Relational symbolic input for RDRL (relational deep reinforcement learning):** Same problem GNN are used to handle variable dim of state and action. since GNN could encode variable-size object and edges
   - Take GNN’s node to predict wheter they are related (e.g. input two node embedding, output score)

> **Note**
> GNN is a powerful encoder providing complex structural inductive bias

## Application-Specific Implementations

The second broad category of articles exploits the versatility of DRL along with the flexible encoding capability of GNNs to address interesting challenges in different application domains

- **Combinatorial Optimization:** Many CO problems are computationally expensive and require approximations and heuristics to solve in polynomial time
   - **Operations Research:** Treat the solution construction as a sequential game. For example, in solving node covering problems, "GNN is first utilized to find appropriate candidate nodes by learning the scoring function computed using the probabilistic greedy approach"
   - **System Design:** Whether it is circuit design or logic synthesis, these are structural optimization tasks. We found that "GCN extracts the circuit features that enable the transfer of knowledge between different topologies", allowing the DRL designer to generalize across different technology nodes.
      - GCN is employed to learn the circuit topology representation. The generalizability of DRL enables training on one technology node and then applying the trained agent to search the same circuit under different technology nodes
- **Transportation:** Transportation problems that are handled with DRL and GNN can be broadly classified into two classes, namely, routing and speed prediction.
   - **Vehicle routing:** The generalization of TSP (one sales man need to visit all city once, and back to the original node). vehicle routing problem has several vehicles, and these vehicles would try to visit all the city with some constraint ****(e.g. lmited travel distance), and return to the original start point.
      - State is the GNN embedding, and the action is to selecting a node from the nonvisited pool and the reward is the negative tour length.
   - **Flow Prediction and Control (TSC):** While routing deals with individual agents, Traffic Signal Control (TSC) deals with the aggregate flow of the system. Traffic is dynamic and heterogeneous. Traditional controllers struggle to adapt to changing topologies or weather.
      - Researchers model traffic networks as heterogeneous graphs where intersections are nodes, and use muti-agent to control.
      - By combining Graph Convolutional Networks (GCN) and Graph Attention Networks (GAT), agents can process spatial dependencies (nearby traffic) and temporal dependencies (flow over time).
      - A Deep Q-Network (DQN) can then act as a meta-controller, providing weights to "combine the GCN and GAT predictions, where the weights are adaptive to different network topologies.”
        
      > Srinivasan, D., Choy, M. C., & Cheu, R. L. (2006). Neural networks for real-time traffic signal control. IEEE Transactions on intelligent transportation systems, 7(3), 261-272.

        
      > **Note**
      > The key contribution of this work lies in its generalizability as it is able to achieve outstanding performance over several real-world networks that are never seen during training.
        
- **Manufacturing and Control:** DRL has also been explored in modern manufacturing systems because of the increasing complexity and interdependency across processes and system levels
   - The manufacturing system is represented as a graph, where machines are treated as nodes
   and material flow between machines is treated as links
      - Use GCN to encode the neighbor informaiton into the machine node serving as the local and the aggregation of whoel graph serving as the global information, and each node represent a agent. (Multi-agent system)
        
      > Huang, Jing, et al. "Integrated process-system modelling and control through graph neural network and reinforcement learning." CIRP Annals 70.1 (2021): 377-380.

   - Connected and Autonomous Vehicles (CAVs)
      - Standard DRL struggles when the number of agents fluctuates. "DRL-based controllers in most existing literature address only a single or fixed number of agents.”
      - The dynamically changing the number of agents (vehicles) and the fast-growing joint action space associated with multiagent driving tasks pose difficultly in achieving cooperative control
      - A centralized multiagent controller is then built upon the fused information to make collaborative decisions for a dynamic number of CAVs within the CAV network.
        
      > Chen, Sikai, et al. "Graph neural network and reinforcement learning for multi‐agent cooperative control of connected autonomous vehicles." Computer‐Aided Civil and Infrastructure Engineering 36.7 (2021): 838-857.

- **Knowledge Representation and Reasoning:** Knowledge Graphs (KGs. task including recommendation systems, social networks, information extraction) store data as triplets (Head, Relation, Tail). However, these databases are often incomplete
   - Usually we predict the link between two entities instead of generating the sub-graph (e.g., new or missing target entities) since the generation task lack of reward signal
   - Advanced models use Generative Adversarial Networks (GANs). An LSTM generator "records previous trajectories... but also generates new subgraphs," allowing the system to reason about entities that might not yet explicitly exist in the graph structure.
- **Life Sciences and Molecular Discovery:** In this domain, the "graphs" are molecules (atoms as nodes, bonds as edges) or biological networks (neurons)
   - Drug Discovery: graph generation
   - Brain Network Analysis: graph analysis

## Problem-Specific Applicability of DRL and GNN Methods

- Sequential decision-making setting of the problem, wherein learning occurs via interactions with the environment
- The learning is aimed at achieving long-term goals and avoids making myopic decisions
- The underlying system is most efficiently represented as a graph, thereby making GNNs the natural choice for representing such system (e.g. TSP problem)
   - Environments involving large graphs should rely on GraphSAGE rather than GCN, as GraphSAGE is a subgraph-based inductive learning approach that is scalable to larger networks.
   - Similarly, applications where the position of a node with respect to the entire graph is vital, PGNNs are preferred. PGNNs explicitly make use of anchor nodes along with neighboring subgraph to improve the effectiveness of node embeddings
    
   > **Note**
   > the expressive power of most GNNs is upper bounded by the 1-Weisfeiler–Lehman (1-WL) graph isomorphism test, i.e., they cannot differentiate between different d-regular graphs
    
- Dynamic graph-structured environments
   - e.g. In a CAV network, the number of vehicles dynamically changes. Under this scenario, an appropriate strategy would be to use LSTMs fused with GNNs for capturing graph evolution (topological changes) as well as DRL trajectories

## OPEN CHALLENGES AND RESEARCH OPPORTUNITIES

<img src="{{ '/assets/notes/drl-gnn-review/image-3.png' | relative_url }}" alt="image 3" loading="lazy" style="max-width: 100%; height: auto;" />

- **Generalizability:** A combined GNN-DRL approach can fall into the pitfall of poor generalizability by not being able to generalize to slightly different environmental settings. For instance, changing the layout of environment
   - e.g., dynamic link connections in social network while solving combinatorial problem such as influence maximization. This can be caused either due to the GNN or the DRL not being able to generalize to the environment.
   - Solution 1: meta RL, provide more context varjable to handle the unseen environments
   - Solution 2: exposing agents to several graph environments
   - Solution 3: decision transformer, leverage the a strong pretrained model and fine tune with specific application
- **Transparency & XAI:** Increasing Transparency by Incorporating XAI and Representation Learning
   - Solution: hierarchical RL / hindsight experience replay
- **Constraint Formulation:** Optimization problems for real-life applications are mostly
bounded by a variety of constraints in terms of finance, time, resources, and so on. Most of the existing DRL works deal with constraints through penalties in reward (soft constraint)
   - when design the MDP or construct the env these constraint need to be considered as a hard constraint (action mask, state transition) to keep the exploration space away from constraint violation
- **Dynamic/Heterogeneous Environment Construction:** real-life scenarios can be far more complex compared to simulated platforms.
   - the real world environment is not static and the entities are not homgenious. how to construct that env and train to handle the dynamics is important.
   - similarly, how to evaluate the model is also important, the model need to be robust. If not, we need to know when to retrain
      - Solution: A sensitivity analysis of the model’s performance to data and environmental parameters can provide important insights into the stability of a model
- **Computational Efficiency via Foundation Decision Models:** RL algorithms are very efficient
during inference time, whereas they are quite sample inefficient (i.e., demand large interaction data for learning an effective policy) during training compared to linear/dynamic programming.

## Related Notes

- Combinatorial Optimization

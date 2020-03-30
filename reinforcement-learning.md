---
layout: page
title: Reinforcement learning
sidebar_link: true
sidebar_sort_order: 1
---

![RL in ML]({{ "/assets/img/ML.png" | relative_url }})

Reinforcement Learning is a subfield of machine learning that teaches an agent how to choose an action from its action space, within a particular environment, in order to maximize rewards over time.

![RL]({{ "/assets/img/RL.png" | relative_url }})

Reinforcement Learning has four essential elements:
- `Agent.` The program you train, with the aim of doing a job you specify.
- `Environment.` The world, real or virtual, in which the agent performs actions.
- `Action.` A move made by the agent, which causes a status change in the environment.
- `Rewards.` The evaluation of an action, which can be positive or negative.

## Supervised, Unsupervised, and Reinforcement Learning: What are the Differences?

### Difference #1: Static Vs.Dynamic
The goal of supervised and unsupervised learning is to search for and learn about patterns in training data, which is quite static. RL, on the other hand, is about developing a policy that tells an agent which action to choose at each step — making it more dynamic.

### Difference #2: No Explicit Right Answer
In supervised learning, the right answer is given by the training data. In Reinforcement Learning, the right answer is not explicitly given: instead, the agent needs to learn by trial and error. The only reference is the reward it gets after taking an action, which tells the agent when it is making progress or when it has failed.

### Difference #3: RL Requires Exploration
A Reinforcement Learning agent needs to find the right balance between exploring the environment, looking for new ways to get rewards, and exploiting the reward sources it has already discovered. In contrast, supervised and unsupervised learning systems take the answer directly from training data without having to explore other answers.

### Difference #4: RL is a Multiple-Decision Process
Reinforcement Learning is a multiple-decision process: it forms a decision-making chain through the time required to finish a specific job. Conversely, supervised learning is a single-decision process: one instance, one prediction.

## Conclusion
- Reinforcement Learning is a subfield of machine learning, parallel to but differing from supervised/unsupervised learning in several ways.
- Because it requires simulated data and environment, it is harder to apply to practical business situations.
- However, its learning process is natural to sequential decision-making scenarios, making RL technology undeniably promising.

There are currently two principal methods often used in RL: 
- probability-based Policy Gradients
- and value-based Q-learning.
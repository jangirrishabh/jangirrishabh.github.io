---
layout: project
title:  "Overcoming exploration from demonstrations in RL"
date:   2018-03-25
types: [all, vision, web-dev, python, opencv, hack, flask, android, scrapy, mongodb]
excerpt: "Implementation of the paper "Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al. over the HER baselines from OpenAI"
tags: [all, projects]
category: code
mathjax: true
comments: true
sourcelink: https://github.com/jangirrishabh/Overcoming-exploration-from-demos
img: research/wamBlockStack.png
---



> OpenAI recently released their Baselines implementation of Hindsight Experience Replay along with a set of [_request for robotics research_](https://blog.openai.com/ingredients-for-robotics-research/). This blog points towards my work in progress for the implementation of the paper ["Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al.](https://arxiv.org/pdf/1709.10089.pdf) which comes under **Combine HER with recent advances in RL** 

There is a vast body of recent research that improves different aspects of RL, and learning from demonstrations has been catching attention in terms of its usage to improve exploration which helps the agent to quickly move to important parts of the state space which is usually large and continuous in most robotics problems. Hindsight Experience Replay (or HER for short), is a reinforcement learning algorithm that can learn from failure, and I highly recommend to read this [paper](https://arxiv.org/abs/1707.01495) before moving ahead with this blog. Also this blog assumes basic understading in Reinforcement learning (off-policy RL algorithms like DQN and DDPG) and Deep Neural Networks.

Abstract from the paper - **Exploration in environments with sparse rewards has been a persistent problem in reinforcement learning (RL). Many tasks are natural to specify with a sparse reward, and manually shaping a reward function can result in suboptimal performance. However, finding a non-zero reward is exponentially more difficult with increasing task horizon or action dimensionality. This puts many real-world tasks out of practical reach of RL methods. In this work, we use demonstrations to overcome the exploration problem and successfully learn to perform long-horizon, multi-step robotics tasks with continuous control such as stacking blocks with a robot arm. Our method, which builds on top of Deep Deterministic Policy Gradients and Hindsight Experience Replay, provides an order of magnitude of speedup over RL on simulated robotics tasks. It is simple to implement and makes only the additional assumption that we can collect a small set of demonstrations. Furthermore, our method is able to solve tasks not solvable by either RL or behavior cloning alone, and often ends up outperforming the demonstrator policy.**

Major contributions of the paper include the following aspects which I have tried to implement over the HER baselines:

### Demonstration Buffer
First, we maintain a second replay buffer R~D~ where we store our demonstration data in the same format as R. In each minibatch, we draw an extra N~D~ examples from R~D~ to use as off-policy replay data  for the update step. These examples are included in both the actor and critic update.
```
self.demo_batch_size = 32 #N~D~ samples

def initDemoBuffer(self, demoDataFile, update_stats=True): #To initiaze the demobuffer with the recorded demonstration data. We also normalize the demo data
        
```


### Behavior Cloning Loss 
Second, we introduce a new loss computed only on the demonstration examples for training the actor

This loss is a standard loss in imitation learning, but we show that using it as an  auxiliary loss for RL improves learning significantly. The gradient applied to the actor parameters is:

(Note  that  we  maximize _J_ and  minimize _L~BC~_ .) Using this loss directly prevents the learned policy from improving significantly beyond the demonstration policy, as the actor is always tied back to the demonstrations.


### Q-Filter
We account for the possibility that demonstrations can be suboptimal  by  applying  the  behavior  cloning  loss  only  to states  where  the  criticQ(s;a) determines  that  the  demon-strator action is better than the actor action


## Training details and Hyperparameters


## Experimentation
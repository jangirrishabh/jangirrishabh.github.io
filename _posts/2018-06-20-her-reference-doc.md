---
layout: project
title:  "Reference doc for HER and Using demonstrations for RL "
date:   2018-06-20
types: [all, Robotics, RL, demonstrations, Gazebo, gym, python, research, machine-learning]
excerpt: "Implementation of the paper Overcoming Exploration in Reinforcement Learning with Demonstrations Nair et al. over the HER baselines from OpenAI"
tags: [all]
mathjax: true
permalink: 2018/06/20/her-reference-doc/
img: research/wam_single_block_reach.png
---

## Problem currently addressing

The problem currently being addressed is of stacking a block on top of another slightly bigger block, the bigger block is already at its desired goal position and the goal position of the smaller block is directly above the goal position of the bigger block. The markers in the following picture show the positions of the goals and current position of the smaller block. 


When the blocks reach their desired goal positions the markers turn green in color. Note that the goal position of the smaller block is only achieved if
- The block is in epsilon(rougly the same size as block) distance of the goal position and,
- The gripper is in a detached state, i.e. it has to release the block at this position for the task to be considered complete

<div class="imgcap">
<center><img src="/assets/research/markers.png" width="80%"></center>
<div class="thecap" align="middle"><b>The Barret WAM robotic arm task visualization in RVIZ.</b></div>
</div>

### Rewards
We give a reward of **0** if the above condition is met, otherwise **-1**. This setting of reward function is the most general case and is know as **Sparse Rewards**.



### Observation space

The observation space of the wam robotic arm is a dictionary which comprises of 3 components - 

```python
self.observation_space = spaces.Dict(dict(
    desired_goal=spaces.Box(-np.inf, np.inf, shape=(6,), dtype='float32'),
    achieved_goal=spaces.Box(-np.inf, np.inf, shape=(6,), dtype='float32'),
    observation=spaces.Box(-np.inf, np.inf, shape=(15,), dtype='float32'),
))
```

namely:
1. observation(length = 15) : [Gripper position in Cartesian(int x3), Gripper orientation(int x4), bool(Gripper state)**Open/Closed**, Big object position(int x3), small object position(int x3), distance b/w small object and gripper]

2. achieved_goal(length = 6) : [Big object position, small object position]

3. desired_goal(length = 6) : [Sampled goal position for big block, Sampled goal position for small block]

### Action space

The action space has 6 continuous components, all in the range [-1, 1] :
Action space - [Move joint 1, move joint 2, Move joint 4, Move joint 5, Move joint 6, Open/close gripper]

Where joint 1, 2 and 4 are for manipulating gripper position and joints 5 and 6 are for manipulating gripper orientation, the last action is for opening and closing the magnetic gripper.

## Demonstration data generation 
With the help of Inverse and Forward kinematics nodes developed at IRI we are commanding the arm to first grasp the block by reaching a point on top of the block and then carrying the block to a position slightly above the goal position and dropping it from there such that it lands on the other slightly bigger block. Note that the demonstrations being generated are not perfect. The following video shows the demonstration generating paradigm.

<div class="imgcap">
<div align="middle">
<iframe width="660" height="400" src="https://www.youtube.com/embed/XHtDISfgXoY? rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>Generating demonstartions for a two block stacking task, note that the demonstrations are not perfect.</b> </div>
</div>

## Method

The implementation is based on using expert demonstrations (that may not be perfect) to move the agent towards important parts of the state space in order to facilitate learning. The basic RL algorithm in cosideration here is called DDPG, Deep Deterministic Policy Gradients, which is an off policy algorithm. So it works by first storing data from interactions with the environment in a replay buffer and then updating the network parameters according to this data, which contains information about the rewards received, actions and observations.

## DDPG

In DDPG we maintain 2 separate networks known as the actor and the critic networks. The critic network is trained to estimate the Q value of a particular action given a state, Q(s,a). And the actor network is trained to output a policy that will give required behavior for the task. These action outputs from the actor are used to generate the environment interactions throughout the training with an added normal noise (10% of max action values) while selecting the actions and selecting a completely random action with a probability of 0.1. The following image shows the DDPG learning framework.


<div class="imgcap">
<center><img src="https://www.stevenspielberg.me/projects/images/ddpg_train.gif" width="600" height="331"></center>
<div class="thecap" align="middle"><b>DDPG learning framework.</b></div>
</div>

## HER
In the case of a sparse reward setting, which is usally easier to use when explaining a complex robotics task, there are not many rollouts with positive rewards. Now even in these failed rollouts where no reward was obtained, the agent can transform them into successful ones by assuming that a state it saw in the rollout was the actual goal. Usually HER is used with any off-policy RL algorithm assuming that for every state we can find a goal corresponding to this state. 

An improvement over the above DDPG algorithm for the **sparse reward** case is using Hindsight Experience replay. In HER we modify the replay buffer by sampling multiple goals in the already present episodes, such that we assume that some of the states achieved in the episode were the goals we were trying to reach and give rewards to the agent for reaching those states. This results in a modified replay buffer with richer information about the task rewards and accelerates learning. 

## Overcoming Exploration with Demonstrations
The above DDPG+HER algorithm works fine, but is fairly slow and requires a lot of time and interations with the environment. The following sections will discuss techniques to use demonstrations from an expert to speed up training and overcome the problem of exploration. We implement 3 techniques as listed below -

### Demonstration Buffer 
First, we maintain a second replay buffer R<sub>D</sub> where we store our demonstration data in the same format as R, the original replay buffer. In each minibatch, we draw an extra N<sub>D</sub> examples from R<sub>D</sub> to use as off-policy replay data  for the update step. These examples are included in both the actor and critic update. Here the total size of our mini-batch is 256 episodes, where 32 episodes are sampled from the demonstration buffer and 224 episodes are sampled from the interaction data buffer.

{% gist bf6f7b5695826fa909aa0bcd85e67e9f %}


### Behavior Cloning Loss
Second, we introduce a new loss computed only on the demonstration examples for training the actor, that is this loss will only be calculated for the episodes from the demonstrations in the sampled mini-batch - 

$$
\begin{align}
L_{BC} &= \sum_{i=1}^{N_D} ||\pi(s_i|\theta_{\pi}) - a_i||^{2}
\end{align}
$$

This loss is a standard loss in imitation learning, but we show that using it as an  auxiliary loss for RL improves learning significantly. The gradient applied to the actor parameters is:

$$
\begin{align}
\lambda_1 \nabla_{\theta_{\pi}} J - \lambda_2 \nabla_{\theta_{\pi}} L_{BC}
\end{align}
$$

(Note  that  we  maximize J and  minimize L<sub>BC</sub>). Where J is the primary loss on the actor parameters given by 

$$
\begin{align}
J = -E_s[Q(s,\pi(s )]
\end{align}
$$

Using this total loss directly prevents the learned policy from improving significantly beyond the demonstration policy, as the actor is always tied back to the demonstrations. 


### Q-value filter 
We account for the possibility that demonstrations can be suboptimal by applying the behavior cloning loss only to states  where  the  critic Q(s,a)  determines  that  the  demonstrator action is better than the actor action. In python this looks like:

{% gist 3cbe9d3eed4695cd8c0e460d58b7a914 %}


Here, we first mask the samples such as to get the cloning loss only on the demonstration samples by using the `tf.boolean_mask` function. We have 3 types of losses depending on the chosen run-paramters. When using both behavior cloning loss with Q_Filter we create another mask that enables us to apply the behavior cloning loss to only those states where the critic Q(s,a) determines that the demonstrator action is better than the actor action.


## Progress




## Tasks
The types of tasks I am considering for now are - 
- [x] Learning Inverse Kinemantics (learning how to reach a particular point inside the workspace)
- [x] Learning to grasp a block and take it to a given goal inside the workspace
- [ ] Learning to stack a block on top of another block
- [ ] Learning to stack 4 blocks on top of each other

> Note: All of these tasks have a sparse reward structure i.e. 0 if the task is complete else a -1.


## Problems currently facing

- Too many hyperparameters and adjustments
- Simulation gets very slow after 3 hours and that causes a problem in learning (Solved!! Moved from ROS actionlib to simple services, actionlib tends to degrade in speed)
- Training on Multiple cores not possibles yet


## Possible improvements

- Recording better demonstrations through VR module being developed by Laia and Sergi
- Attaching an actual gripper to the simulation robot as gripping and releasing is the main problem
- Training on multiple CPUs (In current Gazebo Framework it's hard to run different environments parallely)


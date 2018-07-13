---
layout: project
title:  "Overcoming exploration in RL from demos "
date:   2018-03-25
types: [all, Robotics, RL, demonstrations, Gazebo, gym, python, research, machine-learning]
excerpt: "Implementation of the paper Overcoming Exploration in Reinforcement Learning with Demonstrations Nair et al. over the HER baselines from OpenAI"
tags: [all, research]
category: code
mathjax: true
comments: true
sourcelink: https://github.com/jangirrishabh/Overcoming-exploration-from-demos
img: research/wam_single_block_reach.png
---


There is a vast body of recent research that improves different aspects of RL, and **learning from demonstrations** has been catching attention in terms of its usage to improve exploration which helps the agent to quickly move to important parts of the state space which is usually large and continuous in most robotics problems. Hindsight Experience Replay (or HER for short), is a technique used with reinforcement learning algorithms that can help learn from failure, and I highly recommend to read this [paper](https://arxiv.org/abs/1707.01495) before moving ahead with this blog. Also this blog assumes basic understading in Reinforcement learning (off-policy RL algorithms like DQN and DDPG) and Deep Neural Networks.

> OpenAI recently released their Baselines implementation of Hindsight Experience Replay along with a set of [_request for robotics research_](https://blog.openai.com/ingredients-for-robotics-research/). This blog points towards my work in progress for the implementation of the paper ["Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al.](https://arxiv.org/pdf/1709.10089.pdf) which comes under *Combine HER with recent advances in RL* 


## DDPG
In DDPG we maintain 2 separate networks known as the actor and the critic networks. The critic network is trained to estimate the Q value of a particular action given a state, Q(s,a). And the actor network is trained to output a policy that will give required behavior for the task. These action outputs from the actor are used to generate the environment interactions throughout the training with an added normal noise (10% of max action values) while selecting the actions and selecting a completely random action with a probability of 0.1.


## Hindsight Experience Replay
In the case of a sparse reward setting, which is usally easier to use when explaining a complex robotics task, there are not many rollouts with positive rewards. Now even in these failed rollouts where no reward was obtained, the agent can transform them into successful ones by assuming that a state it saw in the rollout was the actual goal. Usually HER is used with any off-policy RL algorithm assuming that for every state we can find a goal corresponding to this state. 

> Think of trying to shoot a football into a goal, in the unsuccessful tries you hit the ball left to the pole and it does not get inside the goal. But now assume that the goal was originally a little left to where it now is, such that this trial would have been successful in that imaginary case! Now this imaginary case does not help you to learn how to hit the ball exactly in the goal, but you do learn how to hit the ball in a case where the goal was a little to the left, all this was possible because we shifted the goal (to an observed state in the rollout) and gave a reward for it.

HER is a good technique that helps the agent to learn from sparse rewards, but trying to solve robotics tasks with such sparse rewards can be very slow and the agent might not ever reach to some states that are important because of the exploration problem. Think of the case of grasping a block with a robotic arm, the probability of taking the grasp action exactly when the arm position is perfectly above the block in a particular orientation is very low. Thus an improvement to HER would be if we could use expert demonstrations provided to the agent in a way to overcome the exploration problem.

Further in this blog you will read about my implementation of the paper **"Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al.** which introduces ways to use demonstartions along with HER in a sparse reward case to overcome the exploration problems and solve some complex robotics tasks!


## Overcoming Exploration with Demonstrations
The above DDPG+HER algorithm works fine, but is fairly slow and requires a lot of time and interations with the environment. The following sections will discuss techniques to use demonstrations from an expert to speed up training and overcome the problem of exploration. We implement 3 techniques as listed below -

### Demonstration Buffer 
First, we maintain a second replay buffer R<sub>D</sub> where we store our demonstration data in the same format as R. In each minibatch, we draw an extra N<sub>D</sub> examples from R<sub>D</sub> to use as off-policy replay data  for the update step. These examples are included in both the actor and critic update.

{% gist bf6f7b5695826fa909aa0bcd85e67e9f %}


### Behavior Cloning Loss
Second, we introduce a new loss computed only on the demonstration examples for training the actor-

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

> Please read the paper to go through the meaning of the symbols used in these equations

### Q-value filter 
We account for the possibility that demonstrations can be suboptimal by applying the behavior cloning loss only to states  where  the  critic Q(s,a)  determines  that  the  demonstrator action is better than the actor action. In python this looks like:

{% gist 3cbe9d3eed4695cd8c0e460d58b7a914 %}


Here, we first mask the samples such as to get the cloning loss only on the demonstration samples by using the `tf.boolean_mask` function. We have 3 types of losses depending on the chosen run-paramters. When using both behavior cloning loss with Q_Filter we create another mask that enables us to apply the behavior cloning loss to only those states where the critic Q(s,a) determines that the demonstrator action is better than the actor action.

## Experimentation
In [this](https://github.com/jangirrishabh/HER-learn-InverseKinematics) repository I integrated the barret WAM Gazebo simulation with OpenAI gym with the help of [Gym-gazebo](https://github.com/erlerobot/gym-gazebo), thus the simulation environment in gazebo can now be used as a stanalone gym environment with all the functionalities. The plan is to first learn the initial policy on a simulation and then transfer it to the real robot, exploration in RL can lead to wild actions which are not feasible when working with a physical  platform. But currently the whole simulation environment for Barret WAM arm developed at Perception and Manipulation group, IRI is not available in open source, thus I will be reporting the results on Fetch Robotics environments available from OpenAI gym.

### Environments
I'm solving different tasks in two different environments, Fetch robotic environments from OpenAI gym, and Barret WAM simulation in Gazebo integrated with gym. The learning algorithm is agnostic of the simulation environment used. With the help of [Gym-gazebo](https://github.com/erlerobot/gym-gazebo) the simulation environment in gazebo can be used as a stanalone gym environment with all the gym functionalities.
 

<div class="imgcap" align="middle">
<center><img src="/assets/research/wam_single_block_reach.png" ></center>
<div class="thecap" align="middle"><b>The Barret WAM robotic arm simulation in Gazebo.</b></div>
</div>

<div class="imgcap" align="middle">
<center><img src="/assets/research/fetchEnv.png" width="60%"></center>
<div class="thecap" align="middle"><b>The Fetch Arm simulation.</b></div>
</div>

## Tasks
The type of tasks I am considering for now (in Barret WAM) are - 
- [x] Learning Inverse Kinemantics (learning how to reach a particular point inside the workspace)
- [x] Learning to grasp a block and take it to a given goal inside the workspace
- [x] Learning to stack a block on top of another block
- [ ] Learning to stack 4 blocks on top of each other

For the Fetch robotic environments - 
- [x] Reaching
- [x] Pick and Place
- [ ] Push (Difficult to generate demonstrations without VR module)


> Note: All of these tasks have a sparse reward structure i.e. 0 if the task is complete else a -1.

## Generating demonstrations
Currently I am using simple python scripts to generate demonstrations with the help of Inverse IK and Forward IK functionalities already in place for the robot I am using. Thus not all the generated demonstrations are perfect, which is good as our algorithm uses a Q-filter which accounts for all the bad demonstration data. The video below shows the demonstration generating paradigm for one block stacking case, where one of the blocks is already at its goal position and the task involves stacking the second block on top of this block, the goal positions are shown in red in the rviz window next to gazebo (it is way easier to have markers in rviz than gazebo). When the block reaches its goal position the marker turns green.



<div class="imgcap">
<div align="middle">
<iframe width="560" height="315" src="https://www.youtube.com/embed/S2h5S0861DA? rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>Generating demonstartions for a single block stacking task in Barret WAM simulation environment.</b> </div>
</div>


The demonstrations are generated in a similar manner in for the Fetch Pick Place environment, the following video demonstrates the data generation-

<div class="imgcap">
<div align="middle">
<iframe width="560" height="315" src="https://www.youtube.com/embed/pXb2q6VopsE? rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>Generating demonstartions for the Pick and Place task in Fetch robotic environment.</b> </div>
</div>


## Training details and Hyperparameters
We train the robot with the above shown demonstrations in the buffer. We sample a total of 100 demonstrations/rollouts and in every minibatch sample N<sub>D</sub> = 128 samples from the demonstrations in a total of N = 1024 samples, the rest of the samples are generated when the arm interacts with the environment. To train the model we use Adam optimizer with learning rate 0.001 for both critic and actor networks. The discount factor is 0.98. To explore during training, we sample random actions uniformly within the action space with a probability of 0.1 at every step, with an additional uniform gaussian noise which is 10% of the maximum value of each action dimension. For details about more hyperparameters, refer to config.py in the source code. Both the environments are trained with the same set of hyperparameters for now in the reported results.


## Resulting Behaviors
The following video shows the agent's learned behavior corresponding to the task of stacking one block on top of the other. As you can see it learns to pick up the block, reach to a perfect position to drop the block but still does not learn to take the action that results in dropping the block to the goal. As I said this is a work in progress and I am still working towards improving the performace of the agent in this task. For the other easier tasks of reaching a goal position and picking up the block it shows perfect performance, and I have not reported those results as they are just a subset of the current problem which I am trying to solve.

<div class="imgcap">
<div align="middle">
<iframe width="560" height="315" src="https://www.youtube.com/embed/LWiBlc9H0ko? rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>Single block stack task learned bahavior after 1000 epochs on a Barret WAM environment simulation.</b> </div>
</div>

<p></p>

<div class="imgcap">
<div align="middle">
<iframe width="560" height="315" src="https://www.youtube.com/embed/L83qshBkNnU? rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>Pick and place task learned bahavior after 1000 epochs on a Fetch Arm robotic simulation.</b> </div>
</div>

<p></p>

Training with demonstrations helps overcome the exploration problem and achieves a faster and better convergence. The following graphs contrast the difference between training with and without demonstration data, I report the Actor and the Critic network losses vs Epoch, the mean Q values vs Epoch and the Cloning loss vs epoch, note that for the graph without demonstrations the Cloning loss is just a random plot and does not signify anything:



<div class="imgcap">
<center><img src="/assets/research/pickPlaceFetchPart1.png"></center>
<div class="thecap" align="middle"><b>Training results for Fetch Pick and Place task without demonstrations. Actor and Critic losses.</b></div>
</div>

<p></p>

<div class="imgcap">
<center><img src="/assets/research/pickPlaceFetchPart2.png"></center>
<div class="thecap" align="middle"><b>Training results for Fetch Pick and Place task without demonstrations. Cloning loss and mean Q-values.</b></div>
</div>

<p></p>

<div class="imgcap">
<center><img src="/assets/research/fetchPickPlaceWithDemonstrationsPart1.png"></center>
<div class="thecap" align="middle"><b>Training results for Fetch Pick and Place task with the generated demonstrations. Actor and Critic losses.</b></div>
</div>

<p></p>

<div class="imgcap">
<center><img src="/assets/research/fetchPickPlaceWithDemonstrationsPart2.png"></center>
<div class="thecap" align="middle"><b>Training results for Fetch Pick and Place task with the generated demonstrations. Cloning loss and mean Q-values.</b></div>
</div>

<p></p>

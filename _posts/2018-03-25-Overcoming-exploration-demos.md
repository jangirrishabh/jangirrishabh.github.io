---
layout: project
title:  "Overcoming exploration from demonstrations in RL"
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


There is a vast body of recent research that improves different aspects of RL, and **learning from demonstrations** has been catching attention in terms of its usage to improve exploration which helps the agent to quickly move to important parts of the state space which is usually large and continuous in most robotics problems. Hindsight Experience Replay (or HER for short), is a reinforcement learning algorithm that can learn from failure, and I highly recommend to read this [paper](https://arxiv.org/abs/1707.01495) before moving ahead with this blog. Also this blog assumes basic understading in Reinforcement learning (off-policy RL algorithms like DQN and DDPG) and Deep Neural Networks.

> OpenAI recently released their Baselines implementation of Hindsight Experience Replay along with a set of [_request for robotics research_](https://blog.openai.com/ingredients-for-robotics-research/). This blog points towards my work in progress for the implementation of the paper ["Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al.](https://arxiv.org/pdf/1709.10089.pdf) which comes under *Combine HER with recent advances in RL* 


## Hindsight Experience Replay
In the case of a sparse reward setting, which is usally easier to use when explaining a complex robotics task, there are not many rollouts with positive rewards. Now even in these failed rollouts where no reward was obtained, the agent can transform them into successful ones by assuming that a state it saw in the rollout was the actual goal. Usually HER is used with any off-policy RL algorithm assuming that for every state we can find a goal corresponding to this state. 

> Think of trying to shoot a football into a goal, in the unsuccessful tries you hit the ball left to the pole and it does not get inside the goal. But now assume that the goal was originally a little left to where it now is, such that this trial would have been successful in that imaginary case! Now this imaginary case does not help you to learn how to hit the ball exactly in the goal, but you do learn how to hit the ball in a case where the goal was a little to the left, all this was possible because we shifted the goal (to an observed state in the rollout) and gave a reward for it.

HER is a good technique that helps the agent to learn from sparse rewards, but trying to solve robotics tasks with such sparse rewards can be very slow and the agent might not ever reach to some states that are important because of the exploration problem. Think of the case of grasping a block with a robotic arm, the probability of taking the grasp action exactly when the arm position is perfectly above the block in a particular orientation is very low. Thus an improvement to HER would be if we could use expert demonstrations provided to the agent in a way to overcome the exploration problem.

Further in this blog you will read about my implementation of the paper **"Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al.** which introduces ways to use demonstartions along with HER in a sparse reward case to overcome the exploration problems and solve some complex robotics tasks!

Major contributions of the paper include the following aspects which I have tried to implement over the HER baselines:

## Demonstration Buffer
First, we maintain a second replay buffer R<sub>D</sub> where we store our demonstration data in the same format as R. In each minibatch, we draw an extra N<sub>D</sub> examples from R<sub>D</sub> to use as off-policy replay data  for the update step. These examples are included in both the actor and critic update.

```python
self.demo_batch_size = 32 #Number of demo samples

def initDemoBuffer(self, demoDataFile, update_stats=True): 
#To initiaze the demobuffer with the recorded demonstration data. We also normalize the demo data.

def sample_batch(self):
    if self.bc_loss:
        transitions = self.buffer.sample(self.batch_size - self.demo_batch_size)
        global demoBuffer

        transitionsDemo = demoBuffer.sample(self.demo_batch_size)

        for k, values in transitionsDemo.items():
            for v in values:
                rolloutV = transitions[k].tolist()
                rolloutV.append(v.tolist())
                transitions[k] = np.array(rolloutV)
    else:
        transitions = self.buffer.sample(self.batch_size)

```


## Behavior Cloning Loss 
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

(Note  that  we  maximize J and  minimize L<sub>BC</sub>. Using this loss directly prevents the learned policy from improving significantly beyond the demonstration policy, as the actor is always tied back to the demonstrations. 

> Please read the paper to go through the meaning of the symbols used in these equations

## Q-Filter
We account for the possibility that demonstrations can be suboptimal by applying the behavior cloning loss only to states  where  the  critic Q(s,a)  determines  that  the  demonstrator action is better than the actor action. In python this looks like:

```python
self.lambda1 = 0.004
self.lambda2 =  0.031

def _create_network(self, reuse=False):

	mask = np.concatenate((np.zeros(self.batch_size - self.demo_batch_size), np.ones(self.demo_batch_size)), axis = 0)

	target_Q_pi_tf = self.target.Q_pi_tf
    clip_range = (-self.clip_return, 0. if self.clip_pos_returns else np.inf)
    target_tf = tf.clip_by_value(batch_tf['r'] + self.gamma * target_Q_pi_tf, *clip_range) # y = r + gamma*Q(pi)
    self.Q_loss_tf = tf.reduce_mean(tf.square(tf.stop_gradient(target_tf) - self.main.Q_tf)) #(y-Q(critic))^2

    if self.bc_loss ==1 and self.q_filter == 1 :
        maskMain = tf.reshape(tf.boolean_mask(self.main.Q_tf >= self.main.Q_pi_tf, mask), [-1])
        self.cloning_loss_tf = tf.reduce_sum(tf.square(tf.boolean_mask(tf.boolean_mask((self.main.pi_tf / self.max_u), mask), maskMain, axis=0) - tf.boolean_mask(tf.boolean_mask((batch_tf['u']/ self.max_u), mask), maskMain, axis=0)))
        self.pi_loss_tf = -self.lambda1 * tf.reduce_mean(self.main.Q_pi_tf)
        #self.pi_loss_tf += self.lambda1 * self.action_l2 * tf.reduce_mean(tf.square(self.main.pi_tf / self.max_u))
        self.pi_loss_tf += self.lambda2 * self.cloning_loss_tf

    elif self.bc_loss == 1 and self.q_filter == 0:
        self.cloning_loss_tf = tf.reduce_sum(tf.square(tf.boolean_mask((self.main.pi_tf / self.max_u), mask) - tf.boolean_mask((batch_tf['u']/ self.max_u), mask)))
        self.pi_loss_tf = -self.lambda1 * tf.reduce_mean(self.main.Q_pi_tf)
        #self.pi_loss_tf += self.lambda1 * self.action_l2 * tf.reduce_mean(tf.square(self.main.pi_tf / self.max_u))
        self.pi_loss_tf += self.lambda2 * self.cloning_loss_tf

    else:
        self.pi_loss_tf = -tf.reduce_mean(self.main.Q_pi_tf)
        self.pi_loss_tf += self.action_l2 * tf.reduce_mean(tf.square(self.main.pi_tf / self.max_u))
```
Here, we first mask the samples such as to get the cloning loss only on the demonstration samples by using the `tf.boolean_mask` function. We have 3 types of losses depending on the chosen run-paramters. When using both behavior cloning loss with Q_Filter we create another mask that enables us to apply the behavior cloning loss to only those states where the critic Q(s,a) determines that the demonstrator action is better than the actor action.

## Experimentation
The work is in progress and most of the experimentation is being carried out on a Barret WAM simulator, that is because I have access to a Barret WAM robot through the Perception and Manipulation Lab, IRI. I have frameworks for generating demonstartions using the Inverse Kinematics and Forward Kinematics nodes developed at IRI. Also, in [this](https://github.com/jangirrishabh/HER-learn-InverseKinematics) repository I integrated the barret WAM Gazebo simulation with OpenAI gym with the help of [Gym-gazebo](https://github.com/erlerobot/gym-gazebo), thus the simulation environment in gazebo can now be used as a stanalone gym environment with all the functionalities. The plan is to first learn the initial policy on a simulation and then transfer it to the real robot, exploration in RL can lead to wild actions which are not feasible when working with a physical  platform. 

<div class="imgcap">
<center><img src="/assets/research/wam_single_block_reach.png" width="90-%"></center>
<div class="thecap" align="middle">The Barret WAM robotic arm simulation in Gazebo.</div>
</div>

## Tasks
The types of tasks I am considering for now are - 
- [x] Learning Inverse Kinemantics (learning how to reach a particular point inside the workspace)
- [x] Learning to grasp a block and take it to a given goal inside the workspace
- [ ] Learning to stack a block on top of another block
- [ ] Learning to stack 4 blocks on top of each other

> Note: All of these tasks have a sparse reward structure i.e. 0 if the task is complete else a -1.

## Generating demonstrations
Currently using a simple python script to generate demonstrations with the help of Inverse IK and Forward IK functionalities already in place for the robot I am using. Thus not all the generated demonstrations are perfect, which is good as our algorithm uses a Q-filter which accounts for all the bad demonstration data. The video below shows the demonstration generating paradigm for a 2 block stacking case, where one of the blocks is already at its goal position and the task involves stacking the second block on top of this block, the goal positions are shown in red in the rviz window next to gazebo (it is way easier to have markers in rviz than gazebo). When the block reaches its goal position the marker turns green.




<div class="imgcap">
<div align="middle">
<iframe width="600" height="350" src="https://www.youtube.com/embed/XHtDISfgXoY? rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>Generating demonstartions for a two block stacking task, note that the demonstrations are not perfect.</b> </div>
</div>




## Training details and Hyperparameters






## Resulting Behaviors

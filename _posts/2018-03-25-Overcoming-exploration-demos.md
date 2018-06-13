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


There is a vast body of recent research that improves different aspects of RL, and learning from demonstrations has been catching attention in terms of its usage to improve exploration which helps the agent to quickly move to important parts of the state space which is usually large and continuous in most robotics problems. Hindsight Experience Replay (or HER for short), is a reinforcement learning algorithm that can learn from failure, and I highly recommend to read this [paper](https://arxiv.org/abs/1707.01495) before moving ahead with this blog. Also this blog assumes basic understading in Reinforcement learning (off-policy RL algorithms like DQN and DDPG) and Deep Neural Networks.

> OpenAI recently released their Baselines implementation of Hindsight Experience Replay along with a set of [_request for robotics research_](https://blog.openai.com/ingredients-for-robotics-research/). This blog points towards my work in progress for the implementation of the paper ["Overcoming Exploration in Reinforcement Learning with Demonstrations" Nair et al.](https://arxiv.org/pdf/1709.10089.pdf) which comes under *Combine HER with recent advances in RL* 



**Abstract from the original paper** - Exploration in environments with sparse rewards has been a persistent problem in reinforcement learning (RL). Many tasks are natural to specify with a sparse reward, and manually shaping a reward function can result in suboptimal performance. However, finding a non-zero reward is exponentially more difficult with increasing task horizon or action dimensionality. This puts many real-world tasks out of practical reach of RL methods. In this work, we use demonstrations to overcome the exploration problem and successfully learn to perform long-horizon, multi-step robotics tasks with continuous control such as stacking blocks with a robot arm. Our method, which builds on top of Deep Deterministic Policy Gradients and Hindsight Experience Replay, provides an order of magnitude of speedup over RL on simulated robotics tasks. It is simple to implement and makes only the additional assumption that we can collect a small set of demonstrations. Furthermore, our method is able to solve tasks not solvable by either RL or behavior cloning alone, and often ends up outperforming the demonstrator policy.

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
\L_{BC} &= \sum_{i=1}^{N_D} ||\pi(s_i|\theta_{\pi}) - a_i\\^{2}
\end{align}
$$

This loss is a standard loss in imitation learning, but we show that using it as an  auxiliary loss for RL improves learning significantly. The gradient applied to the actor parameters is:

$$
\begin{align}
\lambda_1 \delta_{\theta_{\pi}} \J - \lambda_2 \delta_{\theta_{\pi}} \L_{BC}
\end{align}
$$

(Note  that  we  maximize \\(\J)\\ and  minimize \\(\L_{BC})\\ .) Using this loss directly prevents the learned policy from improving significantly beyond the demonstration policy, as the actor is always tied back to the demonstrations.

## Q-Filter
We account for the possibility that demonstrations can be suboptimal by applying the behavior cloning loss only to states  where  the  critic \\(\Q(s,a))\\  determines  that  the  demonstrator action is better than the actor action. In python this looks like:

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
Here, we first mask the samples such as to get the cloning loss only on the demonstration samples by using the `tf.boolean_mask` function. We have 3 types of losses depending on the chosen run-paramters. When using both behavior cloning loss with Q_Filter we create another mask that enables us to apply the behavior cloning loss to only those states where the critic \\(\Q(s,a))\\ determines that the demonstrator action is better than the actor action.



## Training details and Hyperparameters


## Experimentation

## Results
The work is in progress and most of the experimentation is being carried out on a Barret WAM simulator, that is because I have access to a Barret WAM robot through the Perception and Manipulation Lab, IRI. The plan is to first learn the initial policy on a simulation and then transfer it to the real robot, exploration in RL can lead to wild actions which are not feasible when working with a physical  platform.
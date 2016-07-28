---
layout: post
comments: true
title:  "Apprenticeship learning using Inverse Reinforcement Learning"
excerpt: "Learning distinct behaviors from expert demonstrations by estimating the underlying reward functions, the setting used here is that of a simple agent in a 2D world with differently coloured obstacles"
date:   2016-07-09 22:00:00
mathjax: true
---

**Reinforcement learning (RL)** is is the very basic and most intuitive form of trial and error learning, it is the way by which most of the living organisms with some form of thinking capabilities learn. Often referred to as learning by exploration, it is the way by which a new born human baby learns to take its first steps, that is by taking random actions initially and then slowly figuring out the actions which lead to the forward walking motion. 

> Note, this post assumes a good understanding of the Reinforcement learning framework, please make yourself familiar with RL through week 5 and 6 of this awesome online course [AI_Berkeley](https://www.edx.org/course/artificial-intelligence-uc-berkeleyx-cs188-1x).

Now the question that I kept asking myself is, what is the driving force for this kind of learning, what forces the agent to learn a particular behavior in the way it is doing it. Upon learning more about RL I came across the idea of *rewards*,
basically the agent tries to choose its actions in such a way that the rewards that is gets from that particular behavior are maximized. Now to make the agent perform different behaviors, it is the reward structure that one must modify/exploit. But assume we only have the knowledge of the behavior of the expert with us, then how do we estimate the reward structure given a particular behavior in the environment? Well, this is the very problem of **Inverse Reinforcement Learning (IRL)**, where given the optimal expert policy (actually assumed to be optimal), we wish to determine the underlying reward structure.

<div class="imgcap">
<img src="/assets/IRL/rl_des.png" width="80%">
<div class="thecap" >
	The reinforcement learning framework.
</div>
</div>

<div class="imgcap">
<img src="/assets/IRL/irl_des.png" width="80%">
<div class="thecap" >
	The Inverse reinforcement learning framework.
</div>
</div>

Again, this is not an Intro to Inverse Reinforcement Learning post, rather it is a tutorial on how to use/code Inverse reinforcement learning framework for your own problem, but IRL lies at the very core of it, and it is quintessential to know about it first. IRL has been extensively studied in the past and algorithms have been developed for it, please go through the papers [Ng and Russell,2000](http://ai.stanford.edu/~ang/papers/icml00-irl.pdf), and [Abbeel and Ng, 2004](http://ai.stanford.edu/~ang/papers/icml04-apprentice.pdf) for more information.

> This posts adapts the algorithm from Abbeel and Ng, 2004 for solving the IRL problem.

### Problem to be solved

The idea here is to program a simple agent in a 2D world full of obstacles to copy/clone different behaviors in the environment, the behaviors are input with the help of expert trajectories given manually by a human/computer expert. This form of learning from expert demonstrations is called Apprenticeship Learning in the scientific literature, at the core of it lies inverse Reinforcement Learning, and we are just trying to figure out the different reward functions for these different behaviors.

### Apprenticeship vs. imitation learning - what is the difference?

In general, yes, they are the same thing, which means to learn from demonstration (LfD). Both methods learn from demonstration, but they learn different things:

+ Apprentiship learning via inverse reinforcement learning will try to **infer the goal of the teacher**. In other words, it will learn a reward function from observation, which can then be used in reinforcement learning. If it discovers that the goal is to hit a nail with a hammer, it will ignore blinks and scratches from the teacher, as they are irrelevant to the goal.

+ Imitation learning (a.k.a. behavioral cloning) will try to **directly copy the teacher**. This can be achieved by supervised learning alone. The AI will try to copy every action, even irrelevant actions such as blinking or scratching, for instance, or even mistakes. You could use RL here too, but only if you have a reward function.



### Working Environment

<div class="imgcap">
<img src="/assets/IRL/envo.png" width="100%">
<div class="thecap" >
	The white dots represent the extent to which the agent's sensors extend.
</div>
</div>

+ **Agent:** the agent is a small green circle with its heading direction indicated by a blue line.

+ **Sensors:** the agent is equipped with 3 distance cum color sensors, and this is the only information that the agent has about the environment.

+ **State Space:** the state of the agent consists of 8 observable features- 
1. Distance sensor 1 reading ( /40 to normalize)
2. Distance sensor 2 reading ( /40 to normalize)
3. Distance sensor 3 reading ( /40 to normalize)
4. No. of sensors seeing black color ( /3 to normalize)
5. No. of sensors seeing yellow color ( /3 to normalize)
6. No. of sensors seeing brown color ( /3 to normalize)
7. No. of sensors seeing red color ( /3 to normalize)
8. Boolen to indicate a crash/bump into an obstacle. (1:crash, 0:alive)

>Note, the normalization is done to ensure that every observable feature value is in the range [0,1] which is a necessary condition on the rewards for the IRL algorithm to converge.

+ **Rewards:** the reward after every frame is calculated as a weighted linear combination of the feature values observed in that respective frame. Here the reward \\(r_t\\) in the \\(t^{th}\\) frame is calculated by the dot product of the weight vector \\(w\\) with the vector of feature values in \\(t^{th}\\) frame, that is the state vector \\(\phi_t\\). Such that \\(r_t = w^T*\phi_t\\).

+ **Available Actions:** with every new frame, the agent automatically takes a *forward* step, the available actions can either turn the agent *left*, *right* or *do nothing* that is a simple forward step, note that the turning actions include the forward motion as well, it is not an in-place rotation.

+ **Obstacles:** the environment consists of rigid walls, deliberately colored in different colors. The agent has color sensing capabilities that helps it to dishtinguish between the obstacle types. The environment is designed in this way for easy testing of the IRL algorithm.


+ The Starting position(state) of the bot is fixed, as according to the IRL algorithm it is necessary that the starting state is same for all the iterations.

### Important modifications over the RL algorithm in Matt's code

>Note, that the RL algorithm is completely adopted from [this](https://medium.com/@harvitronix/using-reinforcement-learning in-python-to-teach-a-virtual-car-to-avoid-obstacles-6e782cc7d4c6#.5ylxaemzk) post by Matt Harvey with minor changes, thus it makes perfect sense to talk about the changes that I have made, also even if the reader is comfortable with RL, I highly recommend a glance over that post in order to get an understanding of how the reinforcement learning is taking place.

The environment is significantly changed, with the agent getting abilites to not only sense the distance from the 3 sensors but also sense the color of the obstacles, enabling it to dishtinguish between the obstacles. Also, the agent is now smaller smaller in size and its sensing dots are now closer in order to get more resolution and better performance. Obstacles had to be made static for now, in order to simplify the process of testing the IRL algorithm, this may very well lead to overfitting of data, but I am not concerned about that at the moment. As discussed above the observation set or the state of the agent has been increased from 3 to 8, with the inclusion of the *crash* feature in the agent's state. The reward structure is completely changed, the reward is now a weighted linear combination of these 8 features, the agent no more receives a -500 reward on bumping into obstacles, rather, the feature value for *bumping* is +1 and not bumping is 0 and it is on the algorithm to decide what weight should be assigned to this feature based on the expert behavior.

As stated in Matt's blog, the aim here is to not just teach the RL agent to avoid obstacles, I mean why to assume anything about the reward structure, let the reward structure be completely decided by the algorithm from the expert demonstrations and see what behavior a particular setting of rewards achieves!

### Inverse Reinforcement Learning

#### Important definitions -

1. The **features** or **basis functions** \\(\phi_i\\) which are basically observables in the state. The features in the current problem are discussed above in the state space section. We define \\(\phi(s_t)\\) to be the sum of all the feature expectations \\(\phi_i\\) such that:

	$$
	\begin{align}
	\phi(s_t)  &= \phi_1 + \phi_2 + \phi_3 + ....... + \phi_n \\
	\end{align}
	$$

2. **Rewards** \\(r_t\\) - linear combination of these feature values observed at each state \\(s_t\\).

	$$
	\begin{align}
	r(s,a,s')  &= w_1 \phi_1 + w_2 \phi_2 + w_3 \phi_3 + ....... + w_n \phi_n \\
	&= w^T*\phi(s_t) \\
	\end{align}
	$$

3. **Feature expectations** \\(\mu(\pi)\\) of a policy \\(\pi\\) is the sum of discounted feature values \\(\phi(s_t)\\).

	$$
	\begin{align}
	\mu(\pi) &= \sum_{t=0}^{\infty} \gamma^t \phi(s_t) \\
	\end{align}
	$$

	The feature expectations of a policy are independent of the weights, they only dependent on the states visited during the run (according to the policy) and on the discount factor \\(\gamma\\) a number between 0 and 1 (e.g. 0.9 in our case). To obtain the feature expectations of a policy we have to execute the policy in real time with the agent and record the states visited and the feature values obtained.

#### Initializations

1. Expert policy feature expectations or the **expert's feature expectations** \\(\mu(\pi_E)\\) are obtained by the actions that are taken according to the expert behavior. We basically execute this policy and get the feature expectations as we do with any other policy.
The expert feature expectations are given to the IRL algorithm to find the weights such that the reward funciton corresponding to the weights resemebles the underlying reward function that the expert is trying to maximize (in usual RL language).

2. **Random policy feature expectations** - execute a random policy and use the feature expectations obtained to initialize IRL.

#### The algorithm

1. Maintain a list of the policy feature expectations that we obtain after every iteration.
2. At the very start we have only \\(\pi^1\\) -> the random policy feature expectations.
3. Find the first set of weights of \\(w^1\\) by convex optimization, the problem is similar to an SVM classifier which tries to give a +1 label to the expert feature expec. and -1 label to all the other policy feature expecs.-

	$$
	\begin{align}
	min ||w||_{2}^{2} \\
	\end{align}
	$$

	such that, 

	$$
	\begin{align}
	w^T \mu_E &>= 1 \\ 
	- w^T \mu_{\pi^(i)} &>= 1 \\ \text{definition of expectation}
	\end{align}
	$$

	Termination condition:

	$$
	\begin{align}
	w^T (\mu_E - \mu_{\pi^(i)} ) &<= \epsilon \\ 
	\end{align}
	$$

5. Now, once we get the weights after one iteration of optimization, that is once we get a new reward function, we have to learn the policy which this reward function gives rise to. This is same as saying, find a policy that tries to maximize this obtained reward function. To find this new policy we have to train the Reinforcement Learning algorithm with this new reward function, and train it until the Q-values converge, to get a proper estimate of the policy.

6. After we have learned a new policy we have to test this policy online, in order to get the feature expectations corresponding to this new policy. Then we add these new feature expectaitons to our list of feature expectations and carry on with out IRL algorithm's next iteration until convergence.



### The ones and the zeros

Let us now try to get a hang of the code. Please Find the complete code in [this](https://github.com/jangirrishabh/toyCarIRL) git repo. There are mainly 3 files that you have to worry about - 

+ **manualControl.py** - to get the feature expectations of the expert by manually moving the agent around. Run "python3 manualControl.py", wait for the gui to load and then use your arrow keys to start moving around. Give it the behavior you want it to copy (Note that the behavior which you expect it to copy should be reasonable with the given state space). A good trick would be to assume yourself in place of the agent and think whether you will be able to dishtinguish the given behavior given the current state space only. See the source file for more details.

+ **toy_car_IRL.py** - the main file, this is where the IRL code rests. Let us have a look a the code step by step-

```python
# IRL algorith developed for the toy car obstacle avoidance problem for testing.
import numpy as np
import logging
import scipy
from playing import play #get the RL Test agent, gives out feature expectations after 2000 frames
from nn import neural_net #construct the nn and send to playing
from cvxopt import matrix 
from cvxopt import solvers #convex optimization library
from flat_game import carmunk # get the environment
from learning import IRL_helper # get the Reinforcement learner

NUM_STATES = 8 
BEHAVIOR = 'red' # yellow/brown/red
FRAMES = 100000 # number of RL training frames per iteration of IRL
```
Import dependencies and define the important parameters, change the BEHAVIOR as required. FRAMES are the number of frames you want the RL algorithm to run. 100K is okay and takes around 2 hours.

```python
class irlAgent:
    def __init__(self, randomFE, expertFE, epsilon, num_states, num_frames, behavior):
        self.randomPolicy = randomFE
        self.expertPolicy = expertFE
        self.num_states = num_states
        self.num_frames = num_frames
        self.behavior = behavior
        self.epsilon = epsilon # termination when t < 0.1
        self.randomT = np.linalg.norm(np.asarray(self.expertPolicy)-np.asarray(self.randomPolicy)) #norm of the diff in expert and random
        self.policiesFE = {self.randomT:self.randomPolicy} # storing the policies and their respective t values in a dictionary
        print ("Expert - Random at the Start (t) :: " , self.randomT) 
        self.currentT = self.randomT
        self.minimumT = self.randomT
```
Create the easy to use irlAgent class, which takes in the random and expert behaviors, and the other important parameters as shown.

```python
    def getRLAgentFE(self, W, i): #get the feature expectations of a new poliicy using RL agent
        IRL_helper(W, self.behavior, self.num_frames, i) # train the agent and save the model in a file used below
        saved_model = 'saved-models_'+self.behavior+str(i)+'/164-150-100-50000-'+str(self.num_frames)+'.h5' # use the saved model to get the FE
        model = neural_net(self.num_states, [164, 150], saved_model)
        return  play(model, W)#return feature expectations by executing the learned policy
```
The getRLAgentFE function uses the IRL_helper from the reinforcement learner to train a new model and get feature expectations by playing that model for 2000 iterations. It basically returns the feature expectations for every set of weights (W) it gets.

```python
    def policyListUpdater(self, W, i):  #add the policyFE list and differences
        tempFE = self.getRLAgentFE(W, i) # get feature expectations of a new policy respective to the input weights
        hyperDistance = np.abs(np.dot(W, np.asarray(self.expertPolicy)-np.asarray(tempFE))) #hyperdistance = t
        self.policiesFE[hyperDistance] = tempFE
        return hyperDistance # t = (weights.tanspose)*(expert-newPolicy)
```
To update the dictionary in which we keep our obtained policies and their respective t values.  Where t = (weights.tanspose)*(expert-newPolicy).

```python        
    def optimalWeightFinder(self):
        i = 1
        while True:
            W = self.optimization() # optimize to find new weights in the list of policies
            print ("weights ::", W )
            print ("the distances  ::", self.policiesFE.keys())
            self.currentT = self.policyListUpdater(W, i)
            print ("Current distance (t) is:: ", self.currentT )
            if self.currentT <= self.epsilon: # terminate if the point reached close enough
                break
            i += 1
        return W
```
The implementation of the main IRL algorithm, that is discussed above.

```python    
    def optimization(self): # implement the convex optimization, posed as an SVM problem
        m = len(self.expertPolicy)
        P = matrix(2.0*np.eye(m), tc='d') # min ||w||
        q = matrix(np.zeros(m), tc='d')
        policyList = [self.expertPolicy]
        h_list = [1]
        for i in self.policiesFE.keys():
            policyList.append(self.policiesFE[i])
            h_list.append(1)
        policyMat = np.matrix(policyList)
        policyMat[0] = -1*policyMat[0]
        G = matrix(policyMat, tc='d')
        h = matrix(-np.array(h_list), tc='d')
        sol = solvers.qp(P,q,G,h)

        weights = np.squeeze(np.asarray(sol['x']))
        norm = np.linalg.norm(weights)
        weights = weights/norm
        return weights # return the normalized weights
```
The convex optimization to update the weights upon receving a new policy, basically assign +1 label to expert policy and -1 label to all the other policies and optimize for the weights under the mentioned contraints. To know more about this optimization visit [site](https://courses.csail.mit.edu/6.867/wiki/images/a/a7/Qp-cvxopt.pdf)

```python
if __name__ == '__main__':
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    randomPolicyFE = [ 7.74363107 , 4.83296402 , 6.1289194  , 0.39292849 , 2.0488831  , 0.65611318 , 6.90207523 , 2.46475348]
    # ^the random policy feature expectations
    expertPolicyYellowFE = [7.5366e+00,  4.6350e+00  , 7.4421e+00, 3.1817e-01,  8.3398e+00,  1.3710e-08,  1.3419e+00 ,  0.0000e+00]
    # ^feature expectations for the "follow Yellow obstacles" behavior
    expertPolicyRedFE = [7.9100e+00, 5.3745e-01,  5.2363e+00, 2.8652e+00,  3.3120e+00, 3.6478e-06, 3.82276074e+00  , 1.0219e-17] 
    # ^feature expectations for the follow Red obstacles behavior
    expertPolicyBrownFE = [5.2210e+00,  5.6980e+00,  7.7984e+00,  4.8440e-01, 2.0885e-04, 9.2215e+00, 2.9386e-01 , 4.8498e-17]
    # ^feature expectations for the "follow Brown obstacles" behavior
    epsilon = 0.1
    irlearner = irlAgent(randomPolicyFE, expertPolicyRedFE, epsilon, NUM_STATES, FRAMES, BEHAVIOR)
    print (irlearner.optimalWeightFinder())
```
Create an irlAgent and pass the desired parameters, select between the type of expert behavior you wish to learn the weights for and then run the optimalWeightFinder() function. Note that I have already obtained the feature expectations for red, yellow and brown behaviors.
After the algorithm terminates, you will obtain a list of weights in 'weights-red/yellow/brown.txt', with the respective selected BEHAVIOR. Now, to select the best possible behavior from all the obtained weights, play the saved models in the saved-models_BEHAVIOR/evaluatedPolicies/ directory, the models are saved in the following format **'saved-models_'+ BEHAVIOR +'/evaluatedPolicies/'+iteration number+ '-164-150-100-50000-100000' + '.h5'**. Basically you will get different weights for different iterations, first play the models to find out the model which performs best, then note the iteration number of that model, the weights obtained corresponding to this iteration number are the weights that get you closest to the expert behavior.

+ **playing.py** - to play a model and get the feature expectations after learning a policy and the respective weights, run 'python3 playing.py arg1 arg2 arg3' where arg1 = BEHAVIOR (red/yellow/brown), arg2 = iteration number, arg3 = training frames used in training. This will play the saved model and you can easily evaluate the policy, and to also obtain the feature expectations for this policy you need to replace the weights in the main function of this file.

And then there are files which you probably don't need to update/modify, atleast for the content in this post - 

+ **learning.py** - the reinforcement learning algorithm is implemented here.
+ **flat_game/carmunk.py** - environment/game configuration file.

### The Results, the behaviors and the weights

After around 10-15 iterations the algorithm converges in all the 4 different chosen behaviors, I obtained the following results:


<div class="imgcap">
<div style="display:inline-block">
<iframe width="500" height="360" src="https://www.youtube.com/embed/3PUgmPuJLY4?rel=0&amp;controls=0&amp;autoplay=1&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap"><b>Yellow loving agent: the objective is to go around yellow obstacles without bumping.</b> </div>
</div>

<div class="imgcap">
<div style="display:inline-block">
<iframe width="500" height="360" src="https://www.youtube.com/embed/gFuGes_WNTk?rel=0&amp;controls=0&amp;autoplay=1&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap"><b>Brown loving agent: the objective is to go around brown obstacles without bumping.</b> </div>
</div>

<div class="imgcap">
<div style="display:inline-block">
<iframe width="500" height="360" src="https://www.youtube.com/embed/XXcxL6yq890?rel=0&amp;controls=0&amp;autoplay=1&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap"><b>Red loving agent: the objective is to go around red obstacles without bumping. The boundary is red colored.</b> </div>
</div>

<div class="imgcap">
<div style="display:inline-block">
<iframe width="500" height="360" src="https://www.youtube.com/embed/D64UXmFX4Yc?rel=0&amp;controls=0&amp;autoplay=1&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap"><b>Bumping loving agent: the objective is to bump into obstacles continuously.</b> </div>
</div>



|        Weights           |&nbsp;&nbsp;&nbsp;&nbsp;I love yellow &nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;I love Brown&nbsp;&nbsp;&nbsp;&nbsp;      |&nbsp;&nbsp;&nbsp;&nbsp;I love Red  &nbsp;&nbsp;&nbsp;&nbsp;      |&nbsp;&nbsp;&nbsp;&nbsp;I love Bumping  &nbsp;&nbsp;&nbsp;&nbsp;  |
| :----------------------  |  :------------: |  :------------: |  :------------:  |  :-------------: |
| w1 (Left sensor dist.)   |    -0.0880     |    -0.2627     |     0.2816      |       -0.5892   |
| w2 (middle sensor dist.) |    -0.0624     |     0.0363     |    -0.5547      |       -0.3672   |
| w3 (right sensor dist.)  |    0.0914      |     0.0931     |    -0.2297      |       -0.4660   |
| w4 (Black color)         |    -0.0114     |     0.0046     |     0.6824      |       -0.0299   |
| w5 (yellow color)        |    0.6690      |    -0.1829     |    -0.3025      |       -0.1528   |
| w6 (brown color)         |    -0.0771     |     0.6987     |     0.0004      |       -0.0368   |
| w7 (red color)           |    -0.6650     |    -0.5922     |     0.0525      |       -0.5239   |
| w8 (crash)               |    -0.2897     |    -0.2201     |    -0.0075      |        0.0256   |


#### Possible explanation of the weights - 

+ A high negative value is assigned to the weight which belongs to the *bumping* feature in the first three behaviors, because these 3 expert behaviors do not want the agent to bump into obstacles. Whereas the weight for the same feature in the last behavior, namely the Nasty bot is positive, as the expert behavior advocates bumping.

+ Apparantely the weights for the color features are respective to the expert behavior, high when that color is desired, otherwise a rather low/negative value to get a distinct behavior.

+ The distance feature weights are very ambiguous (counter-intuitive) and it is very difficult to find out some meaningful pattern in the weights. Only thing I wish to point out is that it is even possible to dintinguish clockwise and anti-clockwise behaviors in the current setting, the distance features will carry this information.

> Note, it is very important to first think of whether you as a human will be able to dishtinguish between the given behaviors with the availability of the current state set (the observations) while designing the problem structure. Otherwise you might just be forcing the algorithm to find different weights without providing it the necessary information completely.
	

### Train for a different behavior

If you really want to get into IRL, I would recommend that you actually try to teach the agent a new behavior (you might have to modify the environment for that, as the possible distinct behaviors for the current state set have already been exploited, well atleast according to me).


### Resources and credits - 
1. Thank you Matt Harvey for the game and the RL framework, basically for the father blog to this blog - [Using RL to teach a virtual car to avoid obstacles](https://medium.com/@harvitronix/using-reinforcement-learning-in-python-to-teach-a-virtual-car-to-avoid-obstacles-6e782cc7d4c6#.5ylxaemzk)
2. Andrew Ng and Stuart Russell, 2000 - [Algorithms for Inverse Reinforcement Learning](http://ai.stanford.edu/~ang/papers/icml00-irl.pdf)
3. Pieter Abbeel and Andrew Ng, 2004 - [Apprenticeship Learning via Inverse Reinforcement Learning](http://ai.stanford.edu/~ang/papers/icml04-apprentice.pdf)


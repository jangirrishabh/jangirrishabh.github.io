---
layout: page
comments: true
heading:  "Learning from Demonstration via Inverse Reinforcement Learning."
permalink: /lfD/
mathjax: true
---

<p style="text-align: justify; "> <b>Learning from Demonstration (LfD)</b> is the area of study that talks about teaching complex tasks to agents using expert behaviors given in the form of demonstrations. Two main divisions of this field conflict on whether it is better to learn  a policy (Imitation learning, Behavioral cloning) or is it better to learn the underlying reward structure (Inverse Reinforcement Learning). While inverse reinforcement learning will try to infer the goal of the teacher, imitation learning will try to directly copy the teacher blindly. As pointed out by <b>Abbeel and Ng (2004)</b>, I believe that the reward function, rather than the policy is the more succinct, robust and provides a transferable description of the task. </p>

<p style="text-align: justify; "> <b>Inverse reinforcement learning (IRL)</b> refers to the problem of identifying the underlying reward function for a Markov Decision Process (MDP), given observed, optimal behavior. IRL is useful to ascertain the reward function being optimized by a natural system. A behavior similar to the observed expert behavior is expected to be seen if the MDP is solved using the  obtained reward structure. In order to estimate the desired unknown reward function, we assume that the reward function is parametrized by observable reward features. Usually, these reward features are chosen manually and IRL is used to figure out the weights that perfectly mould the reward function in a way that it advocates the observed behavior.</p>

<p style="text-align: justify; "> My work at the Robotics Research Center, IIIT Hyderabad focussed on <b>learning anti-breakage strategies in Monocular SLAM using Inverse Reinforcement Learning</b>. The idea was to learn a complex task such as low-level robot manoeuvres while preventing monocular SLAM failure, this is a challenging  problem for both robots and humans. The data-driven identification of basic motion strategies in preventing monocular SLAM failure is a largely unexplored problem. I devised a computational model for representing and inferring strategies,  formulated as Markov decision processes, where the reward function models the goal of the task as well as the strategic  information. This reward function can be learnt  from  expert  demonstrations using inverse reinforcement learning. The resulting framework allows  to  identify  the  way  in  which  a  few  chosen  parameters affect  the  quality  of  monocular  slam  estimates.</p>

<!-- <p style="text-align: justify; "> Also, as a part of my research in IRL I created an artificially intelligent agent capable of learning distinct behaviors from expert demonstrations by estimating the underlying reward functions using Inverse Reinforcement Learning . Wrote a <a href="/2016/07/10/virtual-car-IRL/">blog post</a> on the same and released reproduce-able <a href="https://github.com/jangirrishabh/toyCarIRL">code on Github</a> which gained attention in the machine learning community.</p> -->

> Also, as a part of my research in IRL I created an artificially intelligent agent capable of learning distinct behaviors from expert demonstrations by estimating the underlying reward functions using Inverse Reinforcement Learning . Wrote a [blog post](/2016/07/10/virtual-car-IRL/) on the same and released reproduce-able [code on Github](https://github.com/jangirrishabh/toyCarIRL) which gained attention in the machine learning community.

### <u>Learning anti-breakage strategies in Monocular SLAM using Inverse Reinforcement Learning.</u>
<p style="text-align: justify; ">Effective SLAM using a single monocular camera is highly preferred due to its simplicity. However, when compared to trajectory planning methods using depth-based SLAM, Monocular SLAM in loop does need additional considerations. One main reason being that for the optimization, in the form of Bundle Adjustment (BA), to be robust, the SLAM system needs to scan the area for a reasonable duration. Most monocular SLAM systems do not tolerate large camera rotations between successive views and tend to breakdown. Other reasons for Monocular SLAM failure include ambiguities in decomposition of the Essential Matrix, feature-sparse scenes and more layers of non linear optimization apart from BA. </p>

<p style="text-align: justify; ">We present a formulation based on Inverse Reinforcement Learning that generates fail safe trajectories wherein the SLAM generated outputs (scene structure and camera motion) do not deviate largely from their true values. Quintessentially, the IRL framework successfully learns the otherwise complex relation between motor actions and perceptual inputs that result in trajectories that do not cause failure of SLAM, which are almost intractable to capture in an obvious mathematical formulation. We show systematically in simulations how the quality of the SLAM map and trajectory dramatically improves when trajectories are computed by using RL.</p>


> Currently we are in the process of publishing our research on this topic. Here are a few graphics that try to explain our approach.

<div class="imgcap">
<center><img src="/assets/research/lfd3.png" width="50%"></center>
<div class="thecap" ><b>Initially, the agent was trained in a Gazebo simulation, with featured walls to aid monocular SLAM estimates. Later we use this trained data and run it on a real mobile robotic platform (fine-training required).</b></div>
</div>

<div class="imgcap">
<center><img src="/assets/research/lfd2.png" width="50%"></center>
<div class="thecap" ><b>Green represents the ground truth path and blue represents the monocular SLAM predicted trajectory with actions derived from Inverse RL framework.</b></div>
</div> 

<div class="imgcap">
<center><img src="/assets/research/lfd1.png" width="50%"></center>
<div class="thecap" ><p style="text-align: justify; "><b>The algorithm was finally tested on a mobile robotic platform (p3dX) equipped with monocular camera and a laser sensor (for ground truth). We gave expert demonstrations in the form of teleoperation by a human, where the human tries to manouever the robot in such a way that the monocular SLAM does not break. The IRL algorithm learns to even perform better than the human expert. The final results will only be available after publication.</b></p></div>
</div>
----------





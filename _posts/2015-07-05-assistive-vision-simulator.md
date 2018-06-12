---
layout: project
title:  "Assistive Vision Simulator"
date:   2015-07-05
types: [all, vision, web-dev, python, opencv, hack, flask, android, scrapy, mongodb]
excerpt: "Virtual reality environment to test various modalities that enablethe blind to navigate efficiently"
tags: [all, projects]
category: code
mathjax: true
comments: true
img: research/avs2.jpg
---


<p style="text-align: justify; ">A head mounted device with auditory and haptic feedback to better understand and identify the various modalities that can work as substitute to visual information. Our team came up with this idea and developed this product in a span of just 6 days in the REDx Hyderabad workshop'15.</p>
<div class="imgcap">
<center><img src="/assets/research/avs4.jpg" width="70%"></center>
<div class="thecap" align="middle" >Team REDx 2015, Hyderabad.</div>
</div>

### <u>The Hardware</u>
<p style="text-align: justify; ">We modified a pair of headphones by attaching a couple of vibratory motors, a tiny micro-controller and a small Li-ion battery. The listener was continously listening to a mono frequency sound mapped according to the distance of the listeners head from the obstacle in the line of sight. Also the headphones vibrate when the listener is very close to an obstacle or when the listener bumps into an obstacle. Currently the signal to vibrate was being sent serially to the micro-controller, the next step would be to integrate this signal with the audio signal.</p>
<div class="imgcap">
<center><img src="/assets/research/avs2.jpg" width="50%"></center>
<div class="thecap" align="middle">The head mounted device for auditory and haptic feedback. Designed and fabricated by me.</div>
</div>

### <u>The Software</u>
<p style="text-align: justify; ">An Oculus Rift was used for head motion tracking to use in the simulator, only the odometry data of this device was useful to us, as we were working with visually challenged people. A single IMU device attached to the head could have did the job for us but we used an oculus instead due to the time crunch and it also provided us drivers to work with Unity. The simulator environment was developed in Unity, which had integrated drivers for oculus rift. The environment had a basic maze structure with a few other obstacles. </p>
<div class="imgcap">
<center><img src="/assets/research/avs1.jpg" width="50%"></center>
<div class="thecap" align="middle">We developed a VR game in Unity coupled with Oculus Rift.</div>
</div>


### <u>The Testing</u>
<p style="text-align: justify; ">The LVPEI eye care institute helped us to test our system on visually impaired people and get their valuable feedback. This excercise helped us to correct and modify our system to better suit their needs and preferances. Reducing the amplitude of vibration to alarm the listener was one such modification. Working with them we mainly realized that they had better capabilities of identifying the direction of sound, so this system was quite helful and they could navigate through obstacles in the simulator pretty efficiently.</p>
<div class="imgcap">
<center><img src="/assets/research/avs3.jpg" width="50%"></center>
<div class="thecap" align="middle">Our mentor testing the device and the game.</div>
</div>


<iframe class="scribd_iframe_embed" src="https://www.scribd.com/embeds/326035786/content?start_page=1&view_mode=scroll&access_key=key-qmNSrf4PfSQGY4IDFCMb&show_recommendations=true" data-auto-height="false" data-aspect-ratio="0.7066666666666667" scrolling="no" id="doc_46955" width="100%" height="600" frameborder="0"></iframe>


----------

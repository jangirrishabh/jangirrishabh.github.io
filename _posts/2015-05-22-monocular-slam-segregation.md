---
layout: project
title:  "Monocular SLAM supported Point Cloud Segregation of Identified objects"
date:   2015-05-22
types: [all, vision, parrot drone, monocular slam, ptam, pcl, object detection]
excerpt: "PTAM based point cloud segregation system for an identified object in a visual SLAM system"
tags: [all, research]
category: code
mathjax: true
comments: true

img: research/mono1.png
---


<p style="text-align: justify; "> My work at the Robotics Research Lab under Dr. K. Madhava Krishna, was on the Parrot AR.Drone, a quadrocopter with a couple of monocular cameras, I successfully achieved the segregation of the point cloud of a known object from all of the point cloud data available, the object was detected by the drone itself in real time using machine learning algorithms. I worked long with Vignesh Prasad and Sarthak Sharma on Software development and Vision based localization for the AR.Drone.</p>

<div class="imgcap">
<div align="centre">
<iframe width="760" height="428" src="https://www.youtube.com/embed/ram95KVJxMM?rel=0&amp;controls=1&amp;autoplay=0&amp;loop=1&amp;rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>
<div class="thecap" align="middle"><b>The following video shows the method to get a segregated point cloud of an object.</b> </div>
</div>

### <u>Platform</u>
<img style="float:right;margin:0px 0px 0px 20px" src="/assets/research/ardrone2.jpg" width="30%" > 
<p style="text-align: justify; "> The Parrot AR.Drone consists of 2 monocular cameras, one front facing and the otherdownward facing, the downward facing camera is used for stabilization of the drone and velocity estimation and the front facing camera opens up huge possibilities for implementing vision algorithms to achieve autonomous flight adn vision based navigation, originally designed to be just an expensive toy and to record videos on flight, now it has become a popular platform to test vision algorithms for UAVs. It also has a barometer, an ultra-sonic distance sensor, an IMU installed in it. The drone can run continuously for about 12min on its Li-Po battery when fully charged. Due to the lack of a powerful processor all the processing other than the stabilization and execution of commands is done on a separate, more powerful computer to which the drone is connected and should remain connected via WiFi.</p>

### <u>Software and Process Model Development</u>
<p style="text-align: justify; "> The Drone runs the Robot Operating System (ROS). This open architecture permits modularity, and a behavioral approach to the software development. It also facilitates parallel multi-machine processing,and a high level of abstraction of Drone tasks. Packages were developed according to required behaviors. One such package which was developed by Autonomy lab known as the Ardrone Autonomy lets us to establish connection to the drone, access its flight parameters, view the camera feeds andsend control commands to the drone. The Monocular SLAM algorithm in use is known as PTAM or Parallel Tracking and Mapping. This algorithm extracts edges as key-features from the drone camera feed, and outputs a point cloud.</p>


### <u>Object Detection</u>
<p style="text-align: justify; "> The video feed from the front camera is constantly available at the processing station, we use this video feed to implement learning based object detection algorithm Raptor, Raptor is an interactive learning approach for object detection models and a showcase how to quickly perform learning and adaptation with large-scale data sets. It is a framework for quickly training 2D object detectors for robotic perception. We trained raptor   to detect a mug and carried out experiments to get the point cloud of the mug.</p>
<div class="imgcap">
<center><img src="/assets/research/mono1.png" width="60%"></center>
<div class="thecap" align="middle">Detecting a mug in real time.</div>
</div>

### <u>Monocular SLAM framework</u>

<p style="text-align: justify; "> The Technical University of Munich developed a scale-aware navigation algorithm for the ar drone, tum ardrone which is based on PTAM. PTAM or Parallel tracking and mapping is a well know monocular SLAM framework, PTAM extracts key-featuresfrom the available image frame and stores them in key-frames, it does not have to work on all the pixels in the image, thus it is pretty fast and the tracking is also decent. My research was mainly focused on getting a deep insight in the working of this framework and developing algorithms using it. The tum ardrone node publishes pointcloud data of these key-features which acts as the raw data.</p>


<div class="imgcap">
<center><img src="/assets/research/mono3.png" width="60%"></center>
<div class="thecap" align="middle">SLAM Odometry data.</div>
</div>

<div class="imgcap">
<center><img src="/assets/research/mono2.png" width="60%"></center>
<div class="thecap" align="middle">Drone camera Feed and the Monocular SLAM in action.</div>
</div>


### <u>Segregation Method</u>
<p style="text-align: justify; "> The idea is to get a dense point cloud of the detected object by constantly viewing the object at different views, the point cloud of the surrounding has to be ignored so that we get a dense cloud only at the position of the detected object. The raw video feed from the drone is given to the raptor which outputs continuous image frames which contain the detected object, surrounded by a rectangle, these images are then processed by our algorithm. Currently we are controlling the drone manually via a joystick to fly it around the object whose point cloud is to be mapped, this is done to get maximum number of key-points on the object. The raptor outputs an image with a rectangle, the algorithm would save only the key-points which are inside this rectangle and would ignore the others, then we take multiple views and save the point cloud data. Filters from the Point Cloud Library, PCL were then used to get the final segregated point cloud data and saved.</p>

<div class="imgcap">
<center><img  src="/assets/research/mono4.png" width="60%"></center>
<div class="thecap" align="middle">Segregated 3D object.</div>
</div>





------
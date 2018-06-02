---
layout: project
title:  "Wanderlust: Starting Out with Computer Vision"
date:   2014-12-30
types: [all, vision, web-dev, python, opencv, hack, flask, android, scrapy, mongodb]
tags: [all, projects]
category: code
comments: true
sourcelink: https://github.com/anair13/where-am-i
projectlink: http://findwhere.me/
img: wanderlust.png
---

Over winter break, I started learning about computer vision algorithms and working on some intro vision projects. <a href="http://www.amazon.com/gp/product/1449316549/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1449316549&linkCode=as2&tag=ashvinme-20&linkId=2O2Z3UEH7B4KXBYU">This book</a><img src="http://ir-na.amazon-adsystem.com/e/ir?t=ashvinme-20&l=as2&o=1&a=1449316549" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> has been really great for learning about it. It's really clear and introduces both the theory and practice of computer vision.

Anyways, the project: our TreeHacks hack (we built it over 36 hours) is an image-recognition project wrapped into a tour guide application. You take a picture of some landmark around you and it tells you what the landmark is and gives you a history lesson about it. We called it Wanderlust, here's a demo:

<iframe width="100%" height="420px" src="https://www.youtube.com/embed/XPM85kNwkOg" frameborder="0" allowfullscreen></iframe>

I worked on the top level architecture of everything and then implemented the vision system using OpenCV in Python. We have a few different sources of data and images: first, I wrote a scrapy script to grab landmarks and associated locations and descriptions off of the UNESCO world heritage sites page. The locations were passed to the Panoramio API to get a library of images for each landmark. For different cases, we also went around and took pictures of Stanford and grabbed other images manually and tagged them with our own information.

Then the cool stuff: for each image, we run OpenCV feature detectors (SIFT) and pickle these features into MongoDB along with metadata about the image (location, descriptions). To match an image, we compare the features of the image against the library (almost literally a dot product of each feature in each image), and return the associated metadata, which we serve up to the app through a flask REST API. The front-facing Android app POSTs an image to this API and gets back usable information for a map and history blurb.

The android app remains unpublished but the REST api is directly accessible through <a href="http://findwhere.me/">the website</a>, (click through to /wonders/ and try uploading a picture of the Taj Mahal!). For demoing our app, the dataset is small (~200 images) so that it runs in reasonable time, but it scales linearly to whatever size of library we would want. The quality of the image matching is far from perfect; if realtime was not an issue, this would be improved as well.

Moving forward, we got some useful feedback and had some thoughts that would make this viable:

1. we could easily filter for images by a rough idea of your location. Actually, because we're using MongoDB (which supports queries by geolocation) and we already have location data, this would be almost trivial to implement. This means we could have millions of images condensed into features and stuffed into MongoDB, but only have to search through on the order of hundreds to actually match an image.

2. cheat. Instead of the very best matching image, present a few of the best matches and let the user choose which they want to learn more about.

3. moving to PCA-SIFT or some other feature descriptor algorithm might improve the vision system accuracy itself

Fun times! Looking forward to many more image recognition projects in the future.

---
layout: project
title:  "GradeSeer: Student Grade Prediction"
date:   2014-11-30 15:28:00
types: [all, web-dev, machine-learning, rails, angularjs, mongodb, python]
tags: [all, projects]
category: code
comments: true
projectlink: http://104.236.181.205/
sourcelink: https://github.com/anair13/gradebrain
img: gradeseer.png
---

This was our project for CalHacks (go Bears!).

It asks for student grade data, processes it on the backend with MongoDB and Python (currently building multivariate linear regression models), and then stores these models back into MongoDB. Rails uses mongo for its object mapping, and presents these models as an API to the frontend, which renders them in a nice way to the user.

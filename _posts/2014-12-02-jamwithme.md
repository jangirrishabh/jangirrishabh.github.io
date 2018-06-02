---
layout: project
title:  "JamWithMe: In-Browser Music Collaboration"
date:   2014-1-24
types: [all, web-dev, hack, rails, js, firebase]
tags: [all, projects]
category: code
comments: true
projectlink: http://jamwithme.herokuapp.com/
sourcelink: https://github.com/anair13/jam-with-me
img: jamwithme.png
---

A web-app created during HackTech in January 2014, back when modern web development libraries were still quite foreign to me. It uses Firebase to create a collaborative music website where every visitor landing on the website is editing the same piece of music for a minute.

The app uses the Web Audio library in Javascript to play tunes, it actually ended up being quite hard to figure out and use during the 36-hour hackathon! It is engineered to be flexible for complicated sound mixing, but this meant simple playback ended up being pretty annoying. Eventually my hack for playing tones was actually to take a single note from an instrument, then scale it by frequencies corresponding to notes. On the other hand, I found Firebase really great for creating a quick realtime hack.

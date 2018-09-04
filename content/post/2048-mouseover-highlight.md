---
title: 2048 - Mouseover Highlight
date: '2018-07-11T23:15:50-05:00'
draft: false
---
Today, I added code to make the game's restart button light up when the player's mouse is over it. For a while, I was baffled because the `mousemove` event wasn't being triggered when I tested with the developer tools window open. Eventually, I realized that my code works fine when the developer tools panel is closed. I think it might be because the mouse is used for simulating touch events when developer tools are active. 

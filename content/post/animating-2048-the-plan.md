---
title: Animating 2048 - The Plan
date: '2018-07-05T22:05:08-05:00'
draft: false
---
I didn't do much coding today; I was cleaning the apartment and preparing for my upcoming EE 312 exam. However, I have been mulling over how to animate the blocks in 2048.

Here's the strategy I've come up with:

1. In `update()`, make a list of objects being animated with beginning/destination values that are grid coordinate pairs.

1. Pass a start time to `render()`, and have `render()` calculate positions for each image 
using the time elapsed and the distance in pixels between the beginning/destination.

1. Before rendering anything, determine if any animation objects have a destination that is a block. If they do, make sure to draw the destination block as *half* its value (its old value) so that it can promote *after* the incoming block slides in.

1. Related to the previous point: If a moving block is heading toward another block, fade the incoming block out as it approaches its destination (to "slide" the incoming block underneath).

1. With every new `update()` call, clear and regenerate the list of animation objects to skip the previous animation if it's still in progress. This allows the user to animation cancel with rapid, successive key presses. 

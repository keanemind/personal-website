---
title: 2048 in Plain Javascript
date: '2018-07-04T19:28:19-05:00'
draft: false
---
I forgot to update TechLog for the past several days because I have been working non-stop on a project that I started with Bala and Kunal last weekend. We decided to have our own little hackathon, where we would create a project in a non-competitive environment and learn along the way. We chose to make an HTML5 game to learn about client-side web dev. 

Initially, I wanted to use the popular Phaser library to create a more complex game. However, due to resistance from Bala, we opted to use plain Javascript to make something simpler: the once-popular mobile game 2048. We got started by looking up tutorials on making HTML5 games from scratch. We learned how Javascript can render images into an HTML document using the canvas tag, and through trial and error, learned how different Javascript files are linked together by the browser.

By working from Saturday evening to early Sunday morning, we managed to get the game working with simple images, but no animations. The game was inside a simple box on a plain page. Bala wrote most of the game logic, Kunal wrote the rendering function, and I wrote the user input, HTML, and CSS. 

Since Sunday morning, I have been working mostly alone to dramatically change the appearance of the game. The game now looks like it is an app running inside an iPhone X. I added an end game screen, and now I am working on adding animations. Hopefully Bala can help me with this daunting task. 

I have been taking a desktop-first approach to coding the game. So far, testing has mostly been done on large screens. Eventually I am hoping to make the game mobile friendly, but it will take a lot of work to make sure the canvas looks right on devices of various sizes. 

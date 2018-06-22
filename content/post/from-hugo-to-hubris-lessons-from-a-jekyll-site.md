---
title: 'From Hugo to Hubris: Lessons from a Jekyll Site'
date: '2018-06-21T19:12:04-05:00'
draft: false
---
On Tuesday (two days ago), Kunal asked me for help with setting up a personal website for himself. I recommended Jekyll to him for three reasons: 

1. He wanted to host his page on GitHub Pages, and GitHub continuously deploys Jekyll sites automatically.
2. Jekyll's documentation is well organized and well written.
3. Jekyll has tons and tons of themes.

Unfortunately, Kunal approached me for assistance as I was in the middle of [trying to get gdb-dashboard to work](https://techlog.keanenguyen.com/post/living-in-a-shell/). Hoping to retain as many rail cars in my train of thought as possible, I quickly referred him to the Jekyll quick start guide before switching back to reading the gdb-dashboard source code. Kunal promptly sat down next to me and, to my dismay, started watching a YouTube video on how to set up a Jekyll site.

_Is he illiterate!?_ I thought to myself. The quick start barely has 5 steps! 

I realized that I had promised to help him the day before, so I reluctantly closed the code I was looking at and turned to help him. 

"Here, close out of that, I'll help you," I told him. "The first thing we need to do is have Jekyll generate the basic site files."

I hurriedly walked him through creating an empty site and uploading it to a new GitHub repository. I then helped him install the theme he wanted and showed him his new site locally by running the Jekyll server. 

"Let's add a post," I said.

I created a new markdown file in the `_posts` folder, named it something random, then wrote some words in the file. I had Kunal run the `bundle exec jekyll serve` command to test the change, but the build failed. I had never used Jekyll before (honestly, I just assumed everything to be the same as in Hugo) and was unaware that post file names had to follow a certain dated format. Not only that, but I had also completely botched the front matter formatting in my haste. I renamed and edited the file according to the docs and Kunal ran the local server again. This time, the new post showed up right where we expected it; from the home page, we clicked on the link to the posts listing page, and there it was!

I instructed him to add, commit, and push the new post to GitHub. I sighed and waited for the automatic build to complete.

_My work here is done_, I thought._ Back to fixing that damned .gdbinit file!_

Little did I know my impatience would waste far more of my time than it would save!

We quickly realized that, although the build completed successfully, the post was not being listed in the posts page. For the next thirty minutes, we tried all sorts of things. From change the front matter to changing the file name to copying example posts exactly, nothing would work. We disabled caching and refreshed over and over to no avail. For some inexplicable reason, the page always showed up when served locally, but never on GitHub Pages!

I browsed to his site on my computer, went to the Posts page, and grimly confirmed that the post was still missing. I even tried changing the end of the URL to get to the page URL directly. That didn't work either.

Finally, I admitted defeat. It was getting late, and I still needed to wash the dishes. 

"I'm sorry my friend," I said, as I walked to the kitchen sink. "I'll help you with it tomorrow." 

I turned on the water and barely picked up a plate when Kunal said, "Oh! The link to the posts page goes to demo website, not mine!"

I dashed to him.

"My god, you're right! We never set the site URL in the config file!"

I shook my head in disbelief, facepalming internally. _He would have been better off just following that YouTube video!_ I fumed at myself.

Thankfully, editing the config file did the trick and his site was finally fully functional. 

I've learned my lesson. I need to be more patient, and I need to remember: always do the configuration first!

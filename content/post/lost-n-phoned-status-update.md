---
title: Lost-n-Phoned Status Update
date: '2018-06-24T01:01:55-05:00'
draft: false
---
As I went through the Flask documentation on how to deploy to production today, I realized that I am in way over my head. I don't understand how to use a web server with Flask, nor do I know what a WSGI server is. Instead of continuing to work on Lost-n-Phoned, I am going to follow the [tutorial](http://flask.pocoo.org/docs/1.0/tutorial/) in the documentation which walks through making a blogging web app with Flask.

I've skimmed through the tutorial, and I noticed some interesting things. First, it goes over making the whole application a package for quick deployment, which I didn't know was possible. Second, it goes over making automatic tests, which is something I've never done in Python. And finally, it goes over using SQLite3. Upon Googling SQLite, I discovered that it is dramatically different from databases like MySQL! There is no server and no client; just a file on the disk. Perhaps I should have been using it for my [Keane Cogs](https://github.com/keanemind/Keane-Cogs) instead of JSON all along...

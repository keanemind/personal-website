---
title: Deploying Lost-n-Phoned
date: '2018-07-24T21:28:20-05:00'
draft: false
---
Uh oh, I haven't updated this in a long time! That doesn't mean I haven't been pumping out code, though. I've been working hard on preparing [Lost-n-Phoned](https://github.com/ljacob15/lost-and-phoned/) for public use.

A little backstory on Lost-n-Phoned: It's a Flask app that my friends Bala, Leon, and Max made with me at CodeRED Exploration. Essentially, it makes your Google contacts available on any phone through text messaging with the Twilio API.

This past weekend, I packaged Lost-n-Phoned into a pip package for easier deployment. I set up a Digital Ocean droplet and installed the package there. My initial plan was just to use the WSGI server Waitress to run the code. I wanted to run Waitress without privileges, so I set an iptables rule to forward port 80 traffic to the port Waitress was running on. Unfortunately, I quickly discovered that OAuth2 requires a connection over HTTPS, and that Waitress does not support HTTPS at all. 

I decided to go with a standard way of deploying WSGI apps, which is running a WSGI server behind a reverse proxy. Users would connect to the reverse proxy server with HTTPS, and the server would pass the requests along to Waitress through plain HTTP. I wanted to use a certificate from Let's Encrypt for HTTPS, but I didn't want to bother with setting up an ACME client. For that reason, I decided to use Caddy rather than Apache or Nginx for the proxy.

I spent several hours scratching my head at an error Caddy was throwing as it tried to get a certificate. I eventually gave up and installed Nginx. Only when Nginx failed to serve its example site did I finally remember the iptables rule from before! After I removed the rule, Caddy was able to properly use port 80 and ran smoothly.

The next problem I had was with the Flask app thinking that its URL was localhost instead of lostnphoned.com. A bunch of Googling led me to the solution, which was to configure Caddy to pass certain headers in the requests from clients to Waitress. Finally, the Flask app was fully operational.

I spent a few hours today making the lostnphoned.com website with plain HTML and CSS. I went with a simple design because most visitors to the site should be on mobile. I'm pretty happy with how my CSS skills are progressing. The CSS for this site was pretty well-organized and I felt like I understood all the styles I was using.

All that's left now is some more Python coding in the Lost-n-Phoned internals. Two critical features, phone number validation and a security mechanism, are still missing. I'll be back with a progress update soon!

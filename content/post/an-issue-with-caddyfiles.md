---
title: An Issue with Caddyfiles
date: '2018-07-29T20:33:29-05:00'
draft: false
---
I discovered a problem with my Caddyfile today, and considerable Googling and reading of Caddyfile documentation did not help me.

In my Caddyfile, I am proxying the endpoints `/twilio`, `/authorize`, and `/oauth2callback` to my WSGI server. However, one of my static pages was located at `/authorize-success` (it has since been moved to `/success` as a workaround). I was surprised to discover that the folder was captured by the `/authorize` proxy rule! The [Caddy docs](https://caddyserver.com/docs/http-caddyfile#path-matching) suggest using `/authorize/` if I do not want to include `/authorize-success`. When I tried that, Caddy failed to proxy the endpoint properly and I received `404` errors.

I also tried 
```
proxy /authorize {
    except /authorize-success
``` 
and 
```
proxy /authorize {
    except /authorize-success/
```
but both still resulted in `404` errors when I tried to visit authorize-success.

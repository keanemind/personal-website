---
title: Lost-n-Phoned Security
date: '2018-07-29T12:41:10-05:00'
draft: false
---
I have finally, finally, *finally* added a security implementation to Lost-n-Phoned. For now, I'm forcing users to create one-time-use passwords. Every query to the service has to have a password with it, and the password can't be used again. Kunal is strongly opposed to the idea of one-time-use passwords, so I'll discuss the topic with the original CodeRED team and see what they say.

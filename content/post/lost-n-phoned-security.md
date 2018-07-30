---
title: Lost-n-Phoned Security
date: '2018-07-29T12:41:10-05:00'
draft: false
---
I have finally, finally, *finally* added a security implementation to Lost-n-Phoned. For now, I'm forcing users to create one-time-use passwords. Every query to the service has to have a password with it, and the password can't be used again. Kunal is strongly opposed to the idea of one-time-use passwords, so I'll discuss the topic with the original CodeRED team and see what they say.

The implementation is not very complicated. I allow users to add passwords on demand by texting "Add theirpasswordhere". If they have an account, I take the password and store it, salted and hashed, in the database. Unfortunately, as of now, all of each user's passwords are salted with the same salt. In other words, salts are per user instead of per password. Users have a salt generated and stored when the register. I wanted users to be able to input any of their passwords when they make a query. However, if each password had its own salt, I would have had to try each password's salt on the user's input to find a match. For users with more than a few passwords, it would take too long. A compromise had to be made.

I'm very pleased that I got this task out of the way. It had been intimidating me weeks, staring back at me every time I visited the repository's Issues page. Now that it's gone, I can move on to finish one, last, task...

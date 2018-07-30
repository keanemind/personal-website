---
title: Lost-n-Phoned Unique URLs
date: '2018-07-29T18:22:10-05:00'
draft: false
---
Before today, it was possible for people to register a phone number on Lost-n-Phoned even if they did not have access to the phone with that number. They could do this by visiting https://lostnphoned.com/authorize?phone= and appending any phone number.

Why did I set it up this way? The issue was linking HTTP requests by Twilio with HTTP requests by the registering user's web browser. 

Let's go through an example to make this clearer. A user texts the service "Register" to register. The text is received by Twilio and is passed on to my server by an HTTP request coming from Twilio. My server now knows the phone number of the person who wants to register. My server tells Twilio to text the person back with a link to allow Lost-n-Phoned access to their contacts. He clicks the link, but when my server receives the HTTP request from the user's web browser, how is it supposed to know which phone number is associated with this HTTP session? If multiple people are registering at once, which is entirely possible, Flask can't know which HTTP request from Twilio to associate with which HTTP request from the users' web browsers.

This association is important because it is in that second session with the user's web browser that the server receives Google authentication tokens. Those authentication tokens obviously need to be associated with user phone numbers.

My initial solution was to put the user's phone number in a query string. This led to URLs of the form https://lostnphoned.com/authorize?phone=0123456789, as I mentioned earlier. When users clicked the link sent to them, the server used the query string to get the user's phone number. When the user finished authenticating with Google, their number and the tokens from Google were stored together in the database.

Notice, however, that I couldn't verify that the visitor to https://lostnphoned.com/authorize?phone=whatever actually came from the phone with that number! After all, anybody can visit the URL with any phone number in the query string, so anybody can register any phone number.

I tackled this problem by generating a unique URL query string for each registering user. I use the Python uuid module to generate universally unique identifiers (UUIDs). The UUIDs are then converted to base58 strings, which are put in a query string. The result is URLs that look much like the verification links you get in your email. When the user texts "Register", the URL, with the generated query string appended, is sent to the user through Twilio. The query string is also stored in a database along with the user's phone number. Since only the user texting Lost-n-Phoned gets this unique URL, only that user can visit it!

As before, when the user clicks the link, their browser makes a new request to my server. This time, the server uses the UUID in the query string to look up the user's phone number in the database. Now my server could store the number and Google tokens with the guarantee that the user actually texted Lost-n-Phoned from that phone. Voila!

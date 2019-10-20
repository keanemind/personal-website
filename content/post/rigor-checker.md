---
title: Rigor Checker
date: '2019-10-20T00:58:58-05:00'
draft: true
---
[Rigor Checker](https://keanemind.github.io/rigor-checker/)

[GitHub repository](https://github.com/keanemind/rigor-checker)

I built a joke project to determine how rigorous an input mathematical proof is based on the presence of keywords. The website was built with the Google Cloud Vision API, Flask, React, Webpack, and Ant Design.

From the beginning, I knew I wanted to support input via image, PDF, URL, and plain text. I planned on using Google Cloud for the optical character recognition, but did not know how I would support PDFs or URLs.

The first thing I did was build a simple mechanism for turning input text into a numerical "rigor" value. I assigned scores to certain proof-related phrases like "without loss of generality" and "by induction". I then wrote a state machine that used each word from the text as an input to compute the next state. Each state would perform a mathematical operation on the text's current score, which started at 100. [Click here to see the code for this initial implementation](https://github.com/keanemind/rigor-checker/blob/474cf93d17b410b1c17bfa74b5651572781b24b8/rigor_calculator.py).

The next order of business was converting the calculator function I currently had into a REST API and adding a simple UI for inputting text. For the former, I used [Flask](https://palletsprojects.com/p/flask/). For the latter, I chose to try out [Ant Design](https://ant.design/). 

Making the endpoint took no time at all, and I began experimenting with Ant. After adding a text box, I wanted to experiment with what Ant had for file uploads. I knew I wanted mobile users to be able to click and get a prompt to take a photo, so I added `accept='image/*'` to Ant's `Upload.Dragger`. After adding the inputs for PDFs and URLs, I put in some buttons to switch between the input types and gave them a media query to hide their text on smaller devices.

The UI was essentially complete, and I moved back to building the backend. I added a different endpoint for each type of input. This was my first time accepting file uploads from an endpoint, but thankfully, it was very straightforward. I gave Flask a `MAX_CONTENT_LENGTH` configuration to prevent users from uploading oversized files. Using Google Cloud Vision was very simple with their client library. Downloading files from user-provided URLs only took a Google search to figure out.

At this point, I had to figure out PDFs. Another search led me to the [Ghostscript project](https://ghostscript.com/). Instead of bothering with trying to use it as a library, I downloaded the Ghostscript command line tool and used it from Python via `subprocess`. The way I used it, Ghostscript took in a file and outputted a new one. I had to be careful to give Ghostscript's output file a randomly generated filename; it's possible for multiple instances of Ghostscript to run at the same time if multiple requests come in simultaneously because Flask is multithreaded.

However, there was a problem. There are two kinds of PDFs: pure text PDFs and scanned PDFs. If a user provided a scanned PDF, which is basically just a huge picture for every page, Google Cloud Vision should be used on it—not Ghostscript, which just parses text. I did a lot of searching and contemplating. Some people online suggested looking for fonts in the PDF. Others suggested calculating words or images per page. In the end, because I could not find a completely reliable way to determine which kind of PDF a given PDF file was, I chose to disallow scanned PDFs entirely and always used Ghostscript. 

By this time, I had noticed a flaw with my state machine methodology of scanning the text ([see GitHub issue](https://github.com/keanemind/rigor-checker/issues/5)). I took this as a rare opportunity to learn and implement an algorithm. It took me a very long time, but with the help of my friends, I figured out the intuition behind the fairly complicated [Aho-Corasick algorithm](https://en.wikipedia.org/wiki/Aho%E2%80%93Corasick_algorithm) and implemented it. 

Deploying was made painful by the fact that I needed my backend API to be available over HTTPS. To get around this, I ran the Rigor Checker API on the same VPS that [Lost-n-Phoned](https://github.com/ljacob15/lost-and-phoned) ([lostnphoned.com](https://www.lostnphoned.com/)) is running on, since it already had a certificate set up. I had to configure the web server that was reverse proxying to send certain routes to Rigor Checker, but that was all. 

The most satisfying part of this project was being able to visit the website on my phone, click the image upload button, and enter the camera view for taking a picture of a handwritten proof. As soon as I did that and saw the gauge move up to its number, I thought to myself, "this was all worth it."

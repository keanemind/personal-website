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

At this point, I had to figure out PDFs. Another search led me to the [Ghostscript project](https://ghostscript.com/). Instead of bothering with trying to use it as a library, I downloaded the Ghostscript command line tool and used it from Python via `subprocess`.

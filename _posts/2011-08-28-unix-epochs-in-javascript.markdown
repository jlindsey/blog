---
layout: post
date: 2011-08-28
time: 12:09:05 AM
title: Unix Epochs in JavaScript
tags: javascript
---

JavaScript – unlike many of the languages you might use to interact with it – deals with Unix Epoch timestamps in
milliseconds instead of seconds. For instance, PHP's `time()`, Ruby's `Time.now`, and Python's `time.time()` all return the
Epoch in seconds. This can lead to some strange behaviors if you aren't expecting it. It's also fairly annoying when passing
timestamps back and forth between your server-side language and JavaScript.

Here are a few functions you can add to JavaScript's `Date` builtin to make your life easier.

<script src="https://gist.github.com/1176230.js"> </script>


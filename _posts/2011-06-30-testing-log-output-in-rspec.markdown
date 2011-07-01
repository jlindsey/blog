---
layout: post
date: 2011-06-30
time: 07:50:40 PM
title: Testing Log Output in RSpec
tags:
  - rspec
  - tdd
  - ruby
---

In Web Ops, I often find myself writing small daemons or background processors. It's good practice to maintain logs
for these sorts of processes for informational and debugging purposes. Since these logs can be invaluable in
tracking down production errors, it's a good idea to include them in your test suite.

## The Problem

Testing log output can be clumsy and slow. I've seen some tests that attempt to do it by directing their logger to
some temporary file, then reading it back in during the spec. This is problematic for a number of reasons. You have
to write cleanup logic in your test harness to make sure you delete these temporary files. You probably have to
delete them periodically throughout the suite to make sure you don't have any false positives in your checks.

It is, in general, a very clunky solution.

## The Setup

Consider a gem that has a top-level namespace module like so:

<script src="https://gist.github.com/1057540.js?file=example.rb"></script>

It exposes and persists a `Logger` object going to `$stdout`. Fairly straightforward stuff. Let's further say you
have a worker class inside this namespace called `App`: 

<script src="https://gist.github.com/1057540.js?file=app.rb"></script>

Also pretty straightforward. Simply logs a lot as an example.

So now how will we go about testing this log output? Especially the error line in `perform_action`, which we will
need to spec out?

## The Solution

Ruby has got our backs here, with a handy little class called
[StringIO](http://www.ruby-doc.org/stdlib/libdoc/stringio/rdoc/index.html). StringIO acts like a member of Ruby's
`IO` family. It can `read` and `write`, `close` and `open`, `puts` and `flush`. But it's only writing to an
internal String buffer, not a file or other IO sink. This makes it extremely easy to test our log output.

Continuing in our exercise, let's setup our spec helper:

<script src="https://gist.github.com/1057540.js?file=spec_helper.rb"></script>

Here we're setting up a block to be run before each spec which creates a new StringIO buffer and a new Logger with
that buffer, then sets the internal `@@logger` class variable to our Logger instance. We've also created a small
helper function that returns the internal String representation of our StringIO object.

Now we can write the actual spec for our App class:

<script src="https://gist.github.com/1057540.js?file=app_spec.rb"></script>

Successfully testing the output of the Logger.

You can view a project that brings all of this together at my GitHub here:
[https://github.com/jlindsey/log_test_example](https://github.com/jlindsey/log_test_example).


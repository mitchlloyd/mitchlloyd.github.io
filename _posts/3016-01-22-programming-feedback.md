---
layout: post
title:  "Programming Feedback by Example"
date:   3016-01-23
---

<iframe src="https://player.vimeo.com/video/152833900" width="740" height="416" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

Testing and Me
--------------

When I started programming Ruby, passionate testing rhetoric was everywhere but
thoughtful discussion was hard for me to find. The most unmaintainable code
bases I've ever seen covered excruciatingly simple domains and were created by
people that identified as die-hard TDDers. The tests in these projects created
noise and maintenance overhead while providing little feedback. Still, I
realized that automated testing was valuable and I saw glimpses of it working
for open source projects.

When I read Sandi Metz's chapter on "Designing Cost Effective Tests" in
Practical Object Oriented Ruby it was a revelation.

> Most programmers write too many tests... One simple way to get better value
> from tests is to write fewer of them.

I had not heard someone so confidently acknowledge that each test comes with a
cost and articulate the value that tests should provide.

Another testing insight came from Growing Object Oriented Software Guided by
tests. It emphasized that that testing itself is not virtuous. Feedback is the
fundamental tool, and automated testing is just one way to get feedback. When
tests stop providing good feedback, they have lost their value.

Today I have a great fondness for testing, but a high standard for what tests
can live in my projects.

Watching Others Work
--------------------

These screencasts are largely inspired by a series that Justin Searls did called
[My Favorite Way To
TDD](http://blog.testdouble.com/posts/2015-09-10-how-i-use-test-doubles.html).
I've always found it enlightening to watch how other people work and I've
enjoyed all of Justin's talks and blog posts about testing. In his series he
covers an approach to using test doubles that allows him to systematically use a
top-down design approach.

I also hold the idea of top-down design dear, but I've found myself using very
few test doubles recently. As Justin did in his screencasts, I decided to
chronicle an implementation of Conway's Game of Life and reflect on my current
programming approach.

Part 1
------

In Part 1 I begin by creating the UI for a Game of Life application that runs in
a Web Browser.

<iframe src="https://player.vimeo.com/video/152833900" width="740" height="416" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

Parts 2 and 3 are in the works!

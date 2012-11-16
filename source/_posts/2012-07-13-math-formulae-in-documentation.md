---
layout: post
title: "Math formulae in documentation"
description: "Integration of the MathJax library to produce properly typeset formulae on the documentation site."
category:
tags: [documentation, math]
---

This has been something that has provided an unending amount of frustration: creating LaTeX style math
formula on the web. That being said, I'm very thankful to `oscarvarto` on the IRC channel for pointing us
towards a little library called [MathJax](http://www.mathjax.org/ "MathJax").

We will be using this to produce proper documentation for math formulae. I believe that this is
a good step forward for our CIlib documentation effort. As a teaser of the formulae that will
be available, the standard PSO velocity update equation is given below (this includes the
inertia component):

$$v_{ij}(t+1) = w_i v_{ij}(t) + r_1 c_1 (y_{ij}(t) - x_{ij}(t)) + r_2 c_2 (\hat{y}_{j}(t) - x_{ij}(t))$$

This should provide us with better tools going forward :)

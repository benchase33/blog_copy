---
layout: post
output:
  md_document:
    preserve_yaml: true
    variant: markdown_github
subtitle: A Cautionary Demonstration
author: benjamin_chase
tags:
- causal inference
- data mining
readtime: True
last-updated: September 5, 2022
title: The Danger of Wide Nets & Shotgun Regressions
image: https://benchase33.github.io/testing.github.io/assets/conf_effmod_img/dag_3.png
---

 Computers are really good at doing things



**Note**: All source code written for this post is available in a Python
notebook here: <https://colab.research.google.com/drive/1e7WMoYxcrU33WyZTQkIEQd1S9H44trvb?usp=sharing>

<a name="myfootnote1">1</a>: An excellent resource which covers more ways to use propensity scores and alternatives to logistic regression: <https://www.rand.org/pubs/presentations/PT147.html><a href="#footnote-1-ref">&#8617;</a>

<a name="myfootnote2">2</a>: Check out this book for a ground-up explanation of marginal exchangeability and all things related to causal inference: <https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/><a href="#footnote-2-ref">&#8617;</a>

<a name="myfootnote3">3</a>:. An additional resource for understanding and applying propensity score matching: <https://sites.google.com/site/econometricsacademy/econometrics-models/propensity-score-matching><a href="#footnote-3-ref">&#8617;</a>

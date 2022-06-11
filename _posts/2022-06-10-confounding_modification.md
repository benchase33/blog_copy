---
layout: post
output:
  md_document:
    preserve_yaml: true
    variant: markdown_github
subtitle: What’s the Difference and Why Does it Matter?
author: benjamin_chase
tags:
- causal inference
- econometrics
title: Confounding Versus Effect Modification
readtime: True
---

People often want to know if an event causes something to happen. For
instance, you might want to know if undergoing surgery will cause you to
live a longer life. This is typically referred to as a *causal effect*
because an event (surgery) causes something to happen (you live a longer
life). Causal relationships can be represented by a *directed acyclic
graph*, or DAG[^1] for short. Check out the sample DAG below[^2]:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/dag_1.png">
</p>

The arrow going from surgery to lifespan says, in DAG language, that
undergoing surgery has a *direct* causal effect on an individual’s
lifespan. The DAG does not say whether the effect is positive or
negative, only that some causal effect exists. Causal effects can also
be *indirect*. Take a look at an alternative DAG below:

<p align = "center">
<img height = 300px width = auto src="https://benchase33.github.io/testing.github.io/assets/img/dag_2.png">
</p>

This DAG says that undergoing surgery has a direct causal effect on
heart health, and heart health has a direct causal effect on lifespan.
In this DAG, undergoing surgery does not have a direct causal effect on
an individual’s lifespan, but an indirect causal effect *mediated by
heart health*.

The existence of causal effects is often the focus of research. For
instance, scientists likely wanted to know whether or not the COVID-19
vaccine had a causal effect (direct or indirect) on patient outcomes.
Questions like these are typically answered using a *outcome regression
model*[3] in which an outcome (e.g., lifespan) is modeled as a function
of predictors (e.g., vaccine status, obesity, age, etc.). Causal effects
estimated by regression models are heavily dependent on two things:

1.  Which predictors are used in the model.
2.  Which data the model learns from.

If you miss on either point when conducting your analysis, your
conclusions will likely be wrong. These dependencies are the reason that
recognizing the difference between *confounding* and *effect
modification* is crucial to properly estimate causal effects.

## What is Confounding?

I think of confounding as a characteristic a model can have due to bad
choice of predictors. A model should never have confounding when
estimating causal effects because any estimates will be *biased* (this
is a fancy way to say wrong).

A model has confounding when two conditions are met:

1.  There is a *common cause* of the outcome (e.g., lifespan) and the
    thing whose effect we want to estimate (e.g., surgery). This common
    cause is called a *confounder*.
2.  The confounder is **not** properly handled.

In short, confounders almost always exist in data, and models are
plagued with confounding when confounders are not properly handled. Take
a look at the DAG below to see an example of confounding:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/dag_3.png">
</p>

There are three features in this DAG which confirm the presence of a
confounder:

1.  There is an arrow going from weight to surgery, which implies an
    individual’s weight has a direct causal effect on whether they
    undergo surgery.
2.  There is an arrow going from weight to lifespan, which implies an
    individual’s weight has a direct causal effect on their lifespan.
3.  There is **not** an arrow going from surgery to lifespan, which
    implies undergoing surgery does **not** have a direct nor indirect
    causal effect on their lifespan.

Weight is a confounder and needs to be handled properly to estimate the
causal effect of undergoing surgery on lifespan. There are many ways to
do this[^4] [^5] [^6] (matching, IP weighting, standardization,
g-estimation) but a common method is *outcome regression*, which
requires the confounder to be included as a predictor in a linear or
logistic regression model. If the confounder is not included as a
predictor in the model, we have a model with confounding and a biased
estimate of the causal effect.

### Example of Confounding

Let’s look at some sample data to understand how failing to handle
confounders can lead to biased conclusions. Imagine a population in
which only people over a certain weight are eligible for surgery,
heavier people tend to have shorter lifespans, and undergoing surgery
does **not** change an individual’s lifespan. This imaginary universe
reflects the previous DAG with weight as a confounder.

We’ll start with some random data that captures individuals’ weights in
kilograms:

``` py
weight = stats.norm.rvs(loc = 100, scale = 20, size = 1_000)
```

Next, we’ll create surgery data such that only people who weigh over 100
kilograms receive surgery:

``` py
surgery = weight > 100
```

Finally, we’ll create lifespan data such that heavier people have
shorter lifespans:

``` py
lifespan = 100 - weight/10
```

In our sample data, weight has a direct causal effect on both surgery
and lifespan, but surgery does not have a causal effect on weight. We
could build a linear regression model to test whether or not undergoing
surgery has a causal effect on lifespan. Let’s see what happens if we do
not properly handle the confounder, weight, and only include surgery as
a predictor in the model:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/confounding.png">
</p>

We would estimate a significant and negative causal effect of surgery on
lifespan, even though none exists in the data! Remember, we created our
data such that **only** weight impacts lifespan. Failing to include
weight as a predictor here would incorrectly lead us to the conclusion
that undergoign surgery reduces an individual’s lifespan. Now, let’s see
what changes if we properly handle the confounder, weight, and include
it as a predictor in the linear regression model:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/noconfounding.png">
</p>

Now that we properly handled the confounder, we do not estimate a
significant causal effect of surgery on lifespan, and correctly detect
that weight has a significant negative causal effect on lifespan.
Failing to recognize confounding in a model can lead to finding causal
effects that don’t exist, and vice-versa. This is also referred to as
*omitted variable bias* in some fields.

## What is Effect Modification?

While I think of confounding as a problematic characteristic of a model,
I think of effect modification as a neutral characteristic of data.
Models do not have effect modification, but real-world processes do. For
instance, it is possible that undergoing surgery increases an
individual’s lifespan for males but not for females. This is an example
of effect modification because the surgery’s causal on lifespan *depends
on biological sex*. I find the traditional DAG design for effect
modification a bit unintuitive, so check out the DAG below for a
(hopefully) clearer diagram:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/dag_4.png">
</p>

Similar to direct and indirect causal effects, there can be direct and
indirect effect modification. For instance, an individual’s biological
sex might have a causal effect on testosterone, which directly modifies
the effect of surgery. In the DAG below, biological sex is an *indirect
effect modifier* while testosterone is a *direct effect modifier*:

<p align="center">
  <img height="300" src="https://benchase33.github.io/testing.github.io/assets/img/dag_5.png">
</p>

Unlike confounding, effect modification is not a boogeyman that
threatens to invalidate your model and estimated causal effect. Instead,
it can cripple the conclusion of an *imprecise question*. Think about
our original question:

-   Does undergoing surgery have a causal effect on lifespan?

Unfortunately, this question is too vague to properly estimate causal
effects. Instead, we must be more precise. For instance, we could ask:

-   Does undergoing surgery have a causal effect on lifespan **for
    men**?

This is an improvement because our new question better specifies a
population (there is no end to how precise a causal question can be, but
we’ll stop here for now). Let’s walk through an example to see how
effect modification can cause problems.

### Example of Effect Modification

Once again, let’s look at some sample data. We’ll start with a randomly
distributed surgery:

``` py
surgery = np.array([random.randint(0, 2) for _ in range(1_000)])
```

Next, we’ll split the data into two groups, males and females:

``` py
male = np.concatenate([np.zeros(500), np.ones(500)], axis = 0)
```

Finally, we’ll make an outcome variable, lifespan. Lifespan is made such
that surgery has a *positive* causal effect for males and a *negative*
causal effect for females. Therefore, biological sex is a direct effect
modifier.

``` py
lifespan = 75 + 25*surgery*male - 25*surgery*(1 - male)
```

Our original question was to find out whether or not undergoing surgery
will increase an individual’s lifespan. There are no confounders here
because surgery was *randomly assigned*, so we can construct an outcome
regression model with surgery as the only predictor. Let’s see what
happens:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/effectmod.png">
</p>

We would conclude that surgery does not have a significant causal effect
on lifespan in the *entire population*. On average, this is true.
However, this is not true for any *individual* because the surgery has
equal and opposite effects for men and women, whom are evenly
distributed in the population. Due to effect modification paired with
our imprecise question, the causal effects canceled out! Let’s see what
happens when we restrict our model to data for males; this is analogous
to forming a more precise question:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/noeffectmodmale.png">
</p>

With a more precisely specified population (only males), we find a
significant and positive causal effect of surgery on lifespan. We can
see the same for a population with all females:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/noeffectmodfemale.png">
</p>

for whom surgery has a significant and negative causal effect on
lifespan. To be clear, sex is **not** a confounder. It **should not** be
controlled for in the same way as a confounder, such as by including it
as a predictor in our outcome regression model. Let’s see what happens
if we do that:

<p align="center">
  <img height="200" src="https://benchase33.github.io/testing.github.io/assets/img/effectmodbad.png">
</p>

Once again, we do not estimate a significant causal effect of surgery on
lifespan, although it is significant within the respective male and
female populations. We estimate a significant causal effect of sex on
lifespan, however, this is **not** true if nobody in the population
undergoes surgery. The problems associated with an effect modifier
**cannot** be solved by modifying the model, like we did to prevent
confounding. Instead, effect modification requires a more precise
question to avoid misleading conclusions.

## Summary

I think of confounding as a problematic characteristic of a model that
occurs when a confounder is not properly handled. If confounding is not
corrected, causal effects cannot be estimated. Sometimes it is
impossible to properly handle confounders, which is often due to a lack
of data. Techniques, such as *instrumental variables*, exist to combat
this problem, but they all have limitations.

I think of effect modification as a neutral characteristic of data which
is associated with real-world processes. Effect modification cannot be
prevented and should **not** be treated the same way as confounding. If
you are working with a large number of variables, there will almost
always be effect modification to some extent. The best way to avoid
misleading conclusions due to effect modifiers is to make your question
as precise as possible. For example, consider the two possible
questions:

1.  Do email reminders make people save more money?
2.  Does a daily email reminder sent at 10:00PM for 3 weeks, make
    married women, aged 25-34, living in New York City, significantly
    increase their contributions to their IRA?

Confounding and effect modification are two important and distinct
features that should be understood when interpreting reported effects
and doing your own research.

**Note**: All source code written for this post is available in a Python notebook here: https://colab.research.google.com/drive/1Aln4jJfUUQXvK8Fnp75aAMv-utTYoYKp?usp=sharing

[^1]: Directed Acyclic Graphs (DAGs) can be quite complicated when used in practice. You can read more about them and find some additional resources here: https://cran.r-project.org/web/packages/ggdag/vignettes/intro-to-dags.html.

[^2]: I strongly recommend the following Python package for drawing and working with DAGs; it’s the one I used in this post:  https://github.com/ijmbarr/causalgraphicalmodels.

[^3]: Check out this book for a Bayesian approach to linear and logistic regression models: https://www.amazon.com/dp/B09RW8BYQR/ref=cm_sw_em_r_mt_dp_EJDSGXMTW6J9Z2TTZASQ.

[^4]: Check out this book for a deep dive into causal inference techniques: https://mixtape.scunning.com

[^5]: Another resource for causal inference techniques: https://theeffectbook.net

[^6]: A more mathematical resource for causal inference techniques: https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/

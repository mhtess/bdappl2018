---
layout: chapter
title: Why analyze data
description: "Course overview"
---

## Learning about hypotheses from data

The goal of science is to learn about hypotheses or theories through the process of designing experiments and collecting data. 
Hypotheses are unobservable (or, latent): You cannot (in any literal sense) see Newton's equations, but they do generate *predictions* about things you can observe, such as the trajectory of an object through space. 
We are rarely in a position to make *deterministic predictions* (especially concerning social or psychological phenomena): When we hypothesize "Group A will exhibit less free will than Group B", we are making a statistical (or probabilistic) prediction. We mean: "In general, Group A will exhibit ..."
Observing one member of Group B exhibiting more free will than one member of Group A does not falsify the hypothesis. 

To learn about hypotheses from data, you should use statistics. 
There are two primary uses of statistics: For learning about a single hypothesis in isolation (*parameter learning*) and for comparing two or more hypotheses against each other (*hypothesis testing*, or *model comparison*). 
It is argued that hypothesis testing is logically prior to parameter learning (e.g., You want to know that a thing exists before you go out and study its properties), but the logic and computations associated with hypothesis testing are more complex than parameter learning (reft:Wagenmakers2016:benefits; reft:LW2014). 

## Tutorial Learning Goals

**Primary Learning Goal**: Understand Bayesian analyses (e.g., for understanding a paper)

**Practical Goals**:

- Build a simple probabilistic model in a probabilistic programming language
- Perform inference on your simple probabilistic model (to be described later)

## Why Bayesian

- How we think refp:Morey2015:fallacy
- The inferences we would like to draw refp:Wagenmakers2016:benefits
- Tools are developed for flexibility and continuity with cognitive modeling (described below)

## Why use a probabilistic programming language (PPL)

Two reasons: (1) An abstraction barrier between the model and the algorithm for inference (a la `lm` in R) and (2) Flexibility in model development. 

Getting results out of probabilistic models requires *performing Bayesian inference* on the model. 
Probabilistic programming provides an abstraction barrier (i.e., a separation) between the theoretically-interesting *model* and the *inference algorithm* needed to return an answer from the model. 
If the probabilistic language is sufficiently well-developed, one can perform inference on a model even with only cursory knowledge of the algorithm for inference.
We will cover five (common) algorithms in WebPPL, already part of the ecosystem.
You can use them, just by specifying some options.

Using a full-blown probabilistic *language* gives you maximal flexibility for model development and is a common language for both data analysis and cognitive modeling. 
The fundamental strokes of model development are (a) basic programming techniques; (b) a primitive way to define random variables (elementary random primitives or ERPs; a feature of probabilistic languages) and (c) a primitive way to perform inference on a model (the `Infer()` operator in WebPPL).
By employing these fundamental strokes, one can develop increasingly sophisticated models.

Further, one can write model data analytic models and cognitive models in a PPL.
You can even perform a Bayesian data analysis of a Bayesian cognitive model, a so-called *descriptive Bayesian approach* refp:tauber2017descriptive.
For example, Bayesian models of cognition require specifying people's prior; one can infer what the prior knowledge should be given the observed data and the supposed model of cognition. 
If you are able to specify multiple hypotheses in a probabilistic programming language as well as a space of experiments that could potentially disambiguiate the hypotheses, one can automate the search for the optimal experiment in the same ecosystem refp:ouyangwebppl. 
<!-- Cognitive modeling thought of as a separate domain from data analysis: Everybody does data analysis, not everyone models.  -->
<!-- New perspective: AS you start building data analytic models, this leads into cognitive modeling. (At some point, you transition.) -->


### Other languages

In this tutorial, we will use the probabilistic programming language [WebPPL](http://webppl.org). There are many other good programming systems for Bayesian modeling. The two most common are:

- STAN (and now, `brms`)
- BUGS / JAGS

STAN, BUGS, and JAGS are all "restricted languages". They are not full probabilistic languages and thus one cannot specify certain models in those languages. 

In the [next chapter](2-introPPL.html), we will dig into the basics of probabilistic programming.


## References

cite:LW2014

cite:Morey2015:fallacy

cite:tauber2017descriptive

cite:ouyangwebppl

cite:Wagenmakers2016:benefits



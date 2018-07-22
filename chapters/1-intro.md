---
layout: chapter
title: Why analyze data
description: "Course overview"
---

## Learning about hypotheses from data

The goal of science is to learn about hypotheses or theories through the process of designing experiments and collecting data. 
Hypotheses are unobservable (or, latent): You cannot (in any literal sense) see Newton's equations, but they do generate *predictions* about things you can observe, such as the trajectory of an object through space. 
In the social sciences, we are often not in the business of making *deterministic predictions*: When we hypothesize "Group A will exhibit less free will than Group B", we are making a statistical (or probabilistic) prediction (something more akin to "In general, Group A ..."). 
Observing one member of Group B exhibiting more free will than one member of Group A does not falsify the hypothesis. 

Statistics is the glue between hypothesis and data. 
There are two primary uses of statistics: For learning about a single hypothesis in isolation (*parameter learning*) and for comparing two or more hypotheses against each other (*hypothesis testing*, or *model comparison*). 
It is argued that hypothesis testing is logically prior to parameter learning (e.g., You want to know that a thing exists before you go out and study its properties) **cite Wagenmakers**, but the logic and computations associated with hypothesis testing are more complex than parameter learning. 

## Tutorial Learning Goals

Primary Learning Goal: Be able to understand a Bayesian data analysis (e.g., for understanding a paper)

Practical Goals:

- Build a simple probabilistic model in a probabilistic programming language
- Performing inference on your simple probabilistic model (to be described later)

## Why PPL

Fundamental strokes are basic programming techniques with elementary random primitives.

Inference algorithms are the reasons you want to write down models in a probabilisitc programming language. 
Performing inference on a model is possible even with cursory knowledge of the algorithm for inference.
We will cover five (common) algorithms in WebPPL, already part of the ecosystem.
You can use them, just by specifying some options.

### Other languages

## Model as hypothesis

Can develop more sophisticated hypotheses, by employing the fundamental strokes.

Continuity between PP for data analysis and full-blown cogntiive modeling. 
Cognitive modeling thought of as a separate domain from data analysis: Everybody does data analysis, not everyone models. 
New perspective: AS you start building data analytic models, this leads into cognitive modeling. (At some point, you transition.)

With experience, often easy to tell the difference between cog mod and da model. You can in fact do BDA of your BCM, which we will preview at the end. 

OED: Let's say you have 2 hypotheses, instantiated in 2 models. Space of experiments: different conditions, different intensity of stimulus. Models  are mappings from an experimental condition to a set of predictions. Natural to ask: Which experiment would best distinguish the hypotheses?

In the [next chapter](2-introPPL.html), we will dig into the basics of probabilistic programming.
---
layout: chapter
title: Why analyze data
description: "Course overview"
---

## Learning about hypotheses from data


### Parameter inference

### Hypothesis testing

## Course Learning Goals

Primary Learning Goal: Understand Bayesian Data Analysis (e.g., for understanding a paper)

Practical Goals
- Build a simple probabilistic model in a probabilistic programming language
- Performing inference on your simple probabilistic model (to be described later)

## Why PPL

Fundamental strokes are basic programming techniques with elementary random primitives.

Inference algorithms are the reasons you want to write down models in a probabilisitc programming language. 
Performing inference on a model is possible even with cursory knowledge of the algorithm for inference.
We will cover five (common) algorithms in WebPPL, already part of the ecosystem.
You can use them, just by specifying some options.

## Model as hypothesis

Can develop more sophisticated hypotheses, by employing the fundamental strokes.

Continuity between PP for data analysis and full-blown cogntiive modeling. 
Cognitive modeling thought of as a separate domain from data analysis: Everybody does data analysis, not everyone models. 
New perspective: AS you start building data analytic models, this leads into cognitive modeling. (At some point, you transition.)

With experience, often easy to tell the difference between cog mod and da model. You can in fact do BDA of your BCM, which we will preview at the end. 

OED: Let's say you have 2 hypotheses, instantiated in 2 models. Space of experiments: different conditions, different intensity of stimulus. Models  are mappings from an experimental condition to a set of predictions. Natural to ask: Which experiment would best distinguish the hypotheses?

In the [next chapter](2-introPPL.html), we will dig into the basics of probabilistic programming.
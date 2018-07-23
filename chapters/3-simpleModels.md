---
layout: chapter
title: Building models
description: "Describing the generative process of data"
---

In the last chapter, we played around with the basic building blocks of probabilistic programs: sampling from and explicitly representing probability distributions.
In this chapter, we will see how to use probabilistic programs to describe the generative process of data: how the observed data could have been generated.
By explicitly articulating this generative process, we can use Bayesian inference to learn about the latent (or, unobservable) scientifically-interesting parameters of this process.

<!-- Drawing inferences beyond our experimental sample is the domain of statistics. 
If we were only interested in the sample we collect in an experiment (e.g., if you happen you sample the entire population of interest), then there is no need for statistics: You just can describe the data and that would be the end of the story.
But that is not our usual situation: We collect experimental data in order to learn something about the population of interest (and in doing so, we often learn about a hypothesis of interest).
 -->

## A simple example of data analysis

Imagine we are investigating the **origins of prosocial behavior** and are wondering if *16-month-old children* will provide help to a stranger in a given context (for example, as in [this classic study](https://www.youtube.com/watch?v=kfGAen6QiUE) where children observe a stranger struggle to complete a goal).
Note that this sort of binary question: (*do 16-month-olds help?*) is a *hypothesis testing* question. 
Loosely speaking, hypothesis A is that they do help and hypothesis B is that they do not help.
For now, let us put aside the binary question and ask the more continuous question: "What is the propensity of 16-month-olds to help?"
This is still a statistical inference question, since we are not going to measure all 16-month-olds in the world (and even if we could, we still might want to generalize to the 14-month-olds two months in the future...).

Suppose we plan to collect 20 kids worth of data in a single condition of the experiment. (For now, there is no control condition)
Each kid does the single-trial experiment one time, producing either a *helping* (coded as: 1 or `true`) or *not helping* (0 or `false`) behavior.

### Imagining possible outcomes

We can build a probabilistic program to represent a scientific, generative model that can simulate different possible outcomes of our experiment.
(Later, we will use the actual outcome of the experiment to learn about a parameter of this model.)
For our purposes, a generative model is simply one that provides a mapping from latent, unobservable constructs to observable data.
In our example, the latent construct is the *propensity* to help (or, how many 16-months-olds in the whole population would help) and the observable behavior is the *number* of kids in our experiment who help.

The simplest, most idealized model is a `flip()` model: The probability that a kid helps is in proportion to the underlying propensity to help. 
That is, if 16-month-old's have a 0.75 propensity to help, and we collect twenty kids worth of data, we would expect on average 15 of them to help.
To constuct a set of 20 data points, we simply repeat the flip 20 times.

~~~~
// define parameters
var numberOfKidsTested = 20
var propensityToHelp = 0.75

// generate data
var observableResponses = repeat(numberOfKidsTested,
  function(){return flip(probabilityOfHelping)}
)

// summarize data
var numberOfHelpfulResponses = sum(observableResponses)
var observedProportionHelping = numberOfHelpfulResponses/numberOfKidsTested

// display results
display(observableResponses)
display("number of kids who helped = " + numberOfHelpfulResponses)
display("proportion of kids who helped = " + observedProportionHelping)
~~~~

In the code box above, we defined a **generative process** of possible observed data in our experimental setting.
It is a simple generative process, where we assume each participant produces a response *independent* of all others (this is implicit in the `repeat`), and with the same probability (`probabilityOfHelping`).
Note that this assumption underlies a lot of simple frequentist statistics, like a binomial test. 
We might want to relax the first assumption, if for example, all twenty kids were tested simultaneously in the same location and there was the possibility that behavior of one child would influence the other.
We might relax the second assumption if we have more information about each child that we thought might be relevant and/or that is of scientific interest (e.g., demographic variables that we might be interested in like gender or SES, but also *repeated measurements* of the same child).

We represent the data above (`observableResponses`) as an array of Boolean responses (`true` or `false`) as well as the number of `true` (or helpful responses; `numberOfHelpfulResponses`) and the proportion out of 20 (`observedProportionHelping`).

> Exercise: See how changing the `probabilityOfHelping` changes the observed `numberOfHelpfulResponses`. Wrap the lines code into a function, and have it return `numberOfHelpfulResponses`. Use `viz(repeat(1000, newFunction))` to visualize the *distribution* on `numberOfHelpfulResponses`.

This particular generative process (independent flips of a coin) is a very common one and actually has its own primitive distribution that corresponds to the generative process: the `Binomial` distribution.
So, we could rewrite the above code as the following:

~~~~
// define parameters
var numberOfKidsTested = 20
var propensityToHelp = 0.5

// generate data
var numberOfHelpfulResponses = binomial({
  p: propensityToHelp,
  n: numberOfKidsTested
})

// summarize data
var observedProportionHelping = numberOfHelpfulResponses/numberOfKidsTested;

display("number of kids who helped = " + numberOfHelpfulResponses)
display("proportion of kids who helped = " + observedProportionHelping)
~~~~

Note that using the `binomial` distribution loses the information available in `observableResponses` in the code box above. The only additional information in `observableResponses` was the order of the data, which we are assuming is not relevant. (We assume the same model for the 4th participant as we do for the 14th participant.)

It is not always the case that the generative process of the data that we want to assume in our experimental setting has a canonical distribution (like the Binomial) that corresponds to it. But using probabilistic programs, we can represent any generative process by stringing together multiple sampling statements, and possibly transforming them using mathematical (deterministic) operations. We will see this in the next chapter.

### Modeling scientific uncertainty

Above, we looked at the possible outcomes of our experiment assuming some `propensityToHelp`.
But that is not super helpful because we do not know the `propensityToHelp`.
In fact, that is the very thing we would like to learn about!

`propensityToHelp` is a parameter to the distribution that generates the observed data, but it is *latent* (or, unobservable).
The proportion of kids who help in the experiment (`observedProportionHelping`) is an estimate of this latent parameter, but as you can verify above in the code box above, it is not the same thing as `propensityToHelp`.

As scientists, we very often are in the situation of having observed some data in our *sample* and trying to say something about the underlying, generating probability.
That is, we are trying to make *inferences* from the sample to the population.
If children are sensitive to the cues in the our helping experiment, then the `propensityToHelp` could be high.
But if children are not motivated to help (e.g., they don't care about the stranger in our experiment), it could be close to zero.
Figuring out `propensityToHelp` will help us make inferences about the children's sensitivity to our experimental setting, which is ultimately what we're interested in.
How do learn about `propensityToHelp`?

#### Prior distributions over parameters and data

To learn about the `propensityToHelp` from observed data, we must describe our state of knowledge about the latent parameter before having observed any data.

> **Prior distribution**: The state of knowledge of the scientist before any data has been collected.

Sometimes, the *prior distribution* is informed by our expertise as scientists.
Other times, the prior is what we think an objective scientist might assume *a priori* (e.g., a skeptical reviewer).

Determining what the prior distribution should be is one of the intellectual challenges of doing Bayesian data analysis. 
(Indeed, coming to terms with what we truly believe is one of the intellectual challenges of life.) 
Because we do not know what `propensityToHelp` should be, our knowledge must be represented by a probability distribution that assigns (potentially different) probabilities to different possible values of the parameter.
THe task of articulating a prior distribution then reduces to the task of choosing the appropriate distribution and the parameters of that distribution. (As you might imagine, there is potentially no end to this process. What should the parameters of the prior distribution be? Well, if you're not totally sure, you might want to articulate a distribution of possible values for the prior parameters, a so-called *hyperprior* distribution. Often, we don't need to be so extreme in our explicit representation of uncertainty, but it is an important tool to remember that you have in your toolkit.)

Recall that distributions provide both a set of possible values of a variable and their associated probabilities. 
So we can break the problem down into articulating the set of possible values and then the probabilities of each of the values. 

1. **Possible values the parameter could take on**: This is dictated by the role of the parameter in the model.
For example, in abstract terms, `propensityToHelp` is the weight of a coin, and thus must be a number between 0 and 1. (It would not make sense, in the generative model we've described so far, for `propensityToHelp` to be a negative number, or greater than 1.)

2. **Probabilities of all of the possible values of the parameter**: This is much less obvious, but a good starting point is to consider all of the possible values equally probable. That is, we might be completely ignorant as to what the parameter should be (or, we might imagine a host of skeptical reviewers, some of whom think the parameter ought to be high, others who think the parameter ought to be low, and so we intuitively average across all of these potential skeptics). Your understanding of the prior probabilities of the parameter values will grow as you internalize the model more and as we explore the implications of different prior distributions for the results.

So for `propensityToHelp`, a relatively uncontroversial assumption would be that it could be any number between 0 and 1, with all numbers equally likely.
Now that we've made explicit our prior beliefs, we need to find a probability distribution that has these features (if one does not exist, we will need to construct one).
Fortunately, one does exist! 
The [Beta distribution](http://docs.webppl.org/en/master/distributions.html#Beta) is a distribution over the numbers between 0 and 1. 
The Beta distribution has two parameters (denoted in WebPPL as `a` and `b`). 
As can be gleaned from the [Wikipedia article](https://en.wikipedia.org/wiki/Beta_distribution), the parameters that correspond to equal probability for all values between 0 and 1 are `a=1` and `b=1`. 
So our prior distribution is the `Beta({a: 1, b:1})`.


Because assigning equal probabilities to all possible values of a distribution is a very common practice, this distribution can also be described by another family of distributions: The Uniform distribution. The Uniform distribution has two parameters as well (also denoted in WebPPL as `a` and `b`). These parameters are the lower and upper bounds of the range of values. And so, the `Beta({a: 1, b:1})` is equal to `Uniform({a: 0, b:1})`.

~~~~
var PriorDistribution = Uniform({a:0, b:1})

var samplePrior = function(){
  var propensityToHelp = sample(PriorDistribution)
  return propensityToHelp
}

viz(repeat(10000, samplePrior))
~~~~

The `Uniform` distribution (bounded between 0 and 1) represents our *a priori* state of knowledge about `propensityToHelp`.
Specifying what we believe *a priori* is the first step towards determining what we should believe *a posteriori*, or after we observe the data.

We can combine this prior distribution over the parameter with our generative process of the data, which we wrote down above.
Composing these together will produce a prior distribution over observed data.
Distributions over observed data are called **predictive distributions** and so this is the *prior predictive distribution*.
The support of this distribution (i.e., the values over which this distribution is defined) are the possible outcomes of the experiment, whereas the **parameter distribution** above is defined over the values of the latent parameter.

~~~~
var PriorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var samplePriorPredictive = function(){
  var propensityToHelp = sample(PriorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  return numberOfHelpfulResponses
}

viz(repeat(10000, samplePriorPredictive))
~~~~

**Exercise:** How does this distribution differ from the one above it? Why?

#### Learning about parameters from data

Having specified our *a priori* state of knowledge about the parameter of interest, and the generative process of the data given a particular value of the parameter, we are ready to make inferences about the likely value of the parameter given our observed data.
We do this via *Bayesian inference*, the mathematically correct way of reasoning about the underlying probability that generated our data.

So we run the experiment, and 15 out of 20 kids performed the helping behavior.
Thus, `numberOfHelpfulResponses == 15`.
How can we tell our model about this?

~~~~ norun
var sampleAndObserve = function(){
  var propensityToHelp = sample(PriorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  var matchesOurData = (numberOfHelpfulResponses == 15)
  return ...
}
~~~~

What should we return?
We could return `matchesOurData`.
If we repeat this function many times, we will estimate how many `propensityToHelp`s under our prior (i.e., between 0 - 1) give rise to our observed data.
This is called the **likelihood of the data**, but is not immediately interpretable in isolation (though we will see it later in this course).

What if we returned `propensityToHelp`?
Well, that will just give us the same prior that we saw above, because there is no relationship in the program between `matchesOurData` and `propensityToHelp`.

What if we returned `propensityToHelp`, but only if `matchesOurData` is `true`?
In principle, any value `propensityToHelp` *could* give rise to our data, but intuitively some values are more likely to than others (e.g., a `propensityToHelp` = 0.2, would produce 15 out of 20 successes with probability proportional to $$0.2^{15} + 0.8^5$$, which is not as likely as a `propensityToHelp` = 0.8 would have in producing 15 out of 20 success).

It turns out, if you repeat that procedure many times, then the values that survive this "rejection" procedure, survive it in proportion to the actual *a posteriori* probability of those values given the observed data. 
It is a mathematical manifestation of the quotation from Arthur Conan Doyle's *Sherlock Holmes*: "Once you eliminate the impossible, whatever remains, no matter how improbable, must be the truth."
Thus, we eliminate the impossible (and, implicitly, we penalize the improbable), and what we are left with is a distribution that reflects our state of knowledge after having observed the data we collected.

~~~~
var PriorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;

var sampleAndObserve = function(){
  var propensityToHelp = sample(PriorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  var matchesOurData = (propensityToHelp == 15)
  return matchesOurData ? propensityToHelp : "reject"
}

var exampleOutput = repeat(10, sampleAndObserve)

display("___example output from function___")
display(exampleOutput)

// remove all the rejects
var posteriorSamples = filter(
  function(s){return s != "reject" },
  repeat(100000, sampleAndObserve)
)

viz(posteriorSamples)
~~~~

Visualized from the code box above is the *posterior distribution* over the parameter `propensityToHelp`.
It represents our state of knowledge about the parameter after having observed the data (15 out of 20 success).

#### The inference algorithm

The procedure we implemented in the above code box is called [Rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling), and it is the simplest algorithm for [Bayesian inference](https://en.wikipedia.org/wiki/Bayesian_inference).

The algorithm can be written as:

1. Sample a parameter value from the prior (e.g., `p = uniform(0,1)`)
2. Make a prediction (i.e., generate a possible observed data), given that parameter value (e.g., `binomial( {n:20, p: p} )`)
+ If the prediction generates the observed data, record parameter value.
+ If the prediction doesn't generate the observed data, throw away that parameter value.
3. Repeat many times.

Just as we saw in the previous chapter, our ability to represent this distribution depends upon the number of samples we take.
Above, we have chosen to take 100000 samples in order to more accurately represent the posterior distribution.
The number of samples doesn't correspond to anything about our scientific question; it is a feature of the *inference algorithm*, not of our model.
We will describe inference algorithms in more detail in a later chapter.

#### The Posterior Distribution: All you ever wanted

The posterior distribution represents the scientist's state of knowledge after having observed what came out in the experiment. 
Just as with the prior, we can consider both the posterior parameter distribution (what we should believe about the `propensityToHelp`) and the posterior predictive distribution (what we should expect to observe in future experiments).
Though we may think of these as different distributions, they are in actuality just two perspectives on the same posterior distribution. 

Using Bayesian data analysis, scientists can make probability statements about the underlying parameter. 
For example, you can take the posterior distribution and find the points between which 95% of the probability mass is located. 
This is called a Bayesian credible interval. 
(Note that there are actually several related but different ways of creating credible intervals. citet:kruschke2014doing has a helpful discussion of these.)

The following code computes a credible interval in a crude way.
(There are better ways of doing this outside of WebPPL. For example, the `coda` package in R can take a list of samples and estimate a smooth curve for them, and then compute a credible interval.
)

~~~~
var credibleInterval = function(mySamples, credMass){
  var sortedPts = _.sort(mySamples)
  var ciIdxInc = Math.ceil(credMass*sortedPts.length)
  var nCIs = sortedPts.length - ciIdxInc

  var ciWidth = map(function(i){
    sortedPts[i + ciIdxInc] - sortedPts[i]
  },_.range(nCIs))

  var i = _.indexOf(ciWidth, _.min(ciWidth))

  return [sortedPts[i], sortedPts[i+ciIdxInc]]
}

credibleInterval(posteriorSamples, 0.95)
~~~~

The interpretation of a Bayesian credible interval is that "There is a 95% chance that the parameter is between X and Y".
This is in contrast to classical confidence intervals, for which probability statements about parameters are verboten. 
This leads to confusion among practitioners: In one study, the vast majority of undergraduates, masters, and PhDs misunderstood the basic interpretation of a *classical confidence interval*, instead importing a Bayesian interpretation to it citep:Hoekstra2014:misinterpretation.


### Abstracting away from the algorithm: `Infer`

As Bayesian data analysts, we are often in the position of wanting to go from a prior belief distribution to a posterior belief distribution. 
The rejection sampling algorithm we described above is a universal way of doing this (it will always get you the right answer). 
However, it might take an extremely long amount of time to do so (for example, if your observed data is statistically unlikely given your prior). 
Current research in computer science is trying to figure out more and better ways of getting to a posterior distribution.

Probabilistic programming languages provide a useful abstraction barrier that (often) let's the data analyst worry less about the algorithm to get the posterior. This abstraction lets the scientist specify her *generative model of the data* (including: priors, linking function), and the data, and returns to her the posterior. In WebPPL, this abstraction is accomplished by the built-in function `Infer()`.

~~~~
var priorDistribution = Uniform({a:0, b:1});
var numberOfKidsTested = 20;
var model = function() {
  var propensityToHelp = sample(priorDistribution)
  var numberOfHelpfulResponses = binomial({
    p: propensityToHelp,
    n: numberOfKidsTested
  })
  condition(propensityToHelp == 15) // condition on data
  return { propensityToHelp }
}

var posteriorDistibution = Infer({
  model: model, method: "rejection", samples: 1000
})

viz(posteriorDistibution)
~~~~

The output of `Infer()` is a distribution object, in the same way as the built-in distributions like `Binomial()`.

Intuitively, `condition()` here operates the same as the conditional return statement in the code box above this one. 
It takes in a boolean value, and throws out the random choices for which that boolean is `false`. 
Speaking more generally and technically, `condition()` *re-weights* the probabilities of the *program execution* (which includes all of the *random choices* that have been made up to that point in the program) in a binary way: If it's true, the probability of that program execution gets multiplied by 1 (which has no effect) and if the condition statement is false, the probability of that program execution gets multiplied by 0 (which completely destroys that program execution).

`condition()` is a special case of `factor()`, which directly (and continuously) re-weights the (log) probability of the program execution. 
Whereas `condition()` can only take `true` or `false` as arguments, `factor()` takes a number. 
The code above can be rewritten using factor in the following way:


~~~~
var priorDistribution = Uniform({a:0, b:1});

var model = function() {
  var propensityToHelp = sample(priorDistribution)
  // reweight based on log-prob of observing 15
  factor(Binomial( {n:20, p: propensityToHelp} ).score(15))
  return { propensityToHelp }
}
~~~~

Re-weighting the log-probabilities of a program execution by the (log) probability of a value under a given distribution, as is shown in the code box above, is true Bayesian updating. Because this updating procedure is so commonly used, it gets its own helper function: `observe()`.

~~~~
var model = function() {
  var propensityToHelp = uniform(0,1) // priors
  observe(Binomial( {n:20, p: propensityToHelp} ), 15) // observe 15 from the Binomial dist
  return { propensityToHelp }
}
~~~~

#### Observe, condition, and factor

The helper functions `condition()`, `observe()`, and `factor()` all have the same underlying purpose: Changing the probability of different program executions. For Bayesian data analysis, we want to do this in a way that computes the posterior distribution. 

Imagine running a model function a single time. 
In some lines of the model code, the program makes *random choices* (e.g., flipping a coin and it landing on heads, or tails).
The collection of all the random choices in an execution of every line of a program is referred to as the program execution.

Different random choices may have different (prior) probabilities (or perhaps, you have uninformed priors on all of the parameters, and then they each have equal probability).
What `observe`, `condition`, and `factor` do is change the probabilities of these different random choices. 
For Bayesian data analysis, we use these terms to change the probabilities of these random choices to align with the true posterior probabilities. 
For BDA, this is usually achived using `observe`.

`factor` is the most primitive of the three, and `observe` and `condition` are both special cases of `factor`. 
`factor` directly re-weights the log-probability of program executions, and it takes in a single numerical argument (how much to re-weight the log-probabilities). 
`observe` is a special case where you want to re-weight the probabilities by the probability of the observed data under some distribution. `observe` thus takes in two arguments: a distribution and an observed data point.

`condition` is a special case where you want to completely reject or rule out certain program executions.
`condition` takes in a single *boolean* argum

Here is a summary of the three statements.

~~~~ no run
factor(val)
observe(Dist, val) === factor(Dist.score(val)) === condition(sample(Dist) == val)
condition(bool) === factor(bool ? 0 : -Infinity)
~~~~


In the [next chapter](4-bdaFundamentals.html), we'll go through the fundmantals of Bayesian analysis.
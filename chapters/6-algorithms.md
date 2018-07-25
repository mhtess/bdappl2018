---
layout: chapter
title: Inference Algorithms
description: "The various approximate inference algorithms WebPPL provides and the classes of programs for which they are each best suited."
---

### [Enumeration](https://webppl.readthedocs.io/en/master/inference/methods.html#enumeration)

Enumeration essentially "does the math". It builds a big tree of all the possible outcomes and their associated probabilities. It only works if you have ONLY discrete variables, and the more variables you have, the longer it takes.

~~~~
var model = function(){
	var x = uniformDraw(_.range(0, 10))
	var y = uniformDraw(_.range(0, 10))
	condition(x + y > 2)
	return {x, y}
}

var posterior = Infer({model, method: "enumerate"})
viz(posterior)
~~~~

### [Rejection sampling](https://webppl.readthedocs.io/en/master/inference/methods.html#rejection-sampling)

Rejection sampling randomly samples values for all the random variables, check to see if the condition is true, and if it is false, it tries again. Rejection can also work with "soft conditions" (aka factors). The more improbable the condition statement is, the longer it takes to run.

### [MCMC](https://webppl.readthedocs.io/en/master/inference/methods.html#mcmc) (aka Single-site Metropolis-Hastings)

MCMC in WebPPL is a single-site Metropolis Hastings algorithm. It starts by randomly sampling values for all of the variables. Then, it randomly picks one of the variables and resamples its value; before re-running the program, it checks the Metropolis-Hastings (MH) recipe... if MH tells it is a good idea, it accepts the new value, and re-runs the program. If MH indicates it's a bad new value of the variable, it rejects the proposal, and tries to resample a different value. 

The run-time of MH only scales with the number of iterations, a pre-defined value. However, the resuts are only gauranteed to represent the true posterior distribution in the limit. 

### [Incremental MH](https://webppl.readthedocs.io/en/master/inference/methods.html#incremental-mh)

### [Forward](https://webppl.readthedocs.io/en/master/inference/methods.html#forward-sampling)


### Others

- [Variational inference / Optimization](https://webppl.readthedocs.io/en/master/inference/methods.html#optimization)
- [Sequential Monte Carlo (SMC) / Particle Filter](https://webppl.readthedocs.io/en/master/inference/methods.html#smc)
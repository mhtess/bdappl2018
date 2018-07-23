---
layout: chapter
title: Elaborating models
description: "A pinch of sophistication and elegance"
---

You are running a study to examine the efficacy of social messaging on participants' belief in a cause (e.g., eating less meat).
In your study, participants read about the behavior of others in their commmunity and report on their interest in eating less meat (e.g., as in [this paper](https://pdfs.semanticscholar.org/d172/6f274e7c0f7190f83854352d8ffcbda632bb.pdf)). 
Later in the study, they are given the opportunity to donate their monetary compensation to a non-profit that supports the cause. 

## Inferring a rate

You run your study on facebook and collect 1000 participants worth of data.
Your data set includes the messaging condition (experimental vs. control), the amount of time they spent on the experiment, their rating of their belief in the cause, whether or not the participant donated their earnings, and miscellaneous demographic data (their browser, location).

Let's take a look at the data.

~~~~
///fold:
var head = function(ar, l){
	var len = l ? l : 6;
	return print(ar.slice(0, len));
}
///
 
// print first few lines of data set
head(bannerData)

// show all the ids
print(_.pluck(bannerData, "id"))

// how many people saw each banner?
viz.table(_.pluck(bannerData, "condition"))
~~~~

Your main hypothesis is that the donation rate is going to be higher for the group in the experimental condition than in the control condition. 

~~~~
///fold:
var foreach = function(lst, fn) {
    var foreach_ = function(i) {
        if (i < lst.length) {
            fn(lst[i]);
            foreach_(i + 1);
        }
    };
    foreach_(0);
};
///

var model = function() {

  // unknown rates of donation
  var donationRates = {
    grey: uniform(0,1),
    green: uniform(0,1)
  };

  foreach(bannerData, function(personData) {

      // grab appropriate conversionRate by condition
      var acceptanceRate = conversionRates[personData["condition"]];

      // visitors are i.i.d.
      observe(Bernoulli({p:acceptanceRate}), personData["converted"])

  });

  return _.extend(conversionRates, 
                  {delta: conversionRates.green - conversionRates.grey});

}

var numSamples = 10000;
var inferOpts = {
  method: "MCMC", 
  samples: numSamples,
  burn: numSamples/2, 
  callbacks: [editor.MCMCProgress()] 
};

var posterior = Infer(inferOpts, model);

viz.marginals(posterior)	
~~~~

1. Scientists often use a 95% threshold to say whether or not an inferred difference is "real". In this example, we would draw what's called a 95% *credible interval* (an interval in which there is a 95% probability that the true parameter is contained), and determine whether or not there is a real difference by seeing if this interval includes 0. By examining the figure, what can you conclude about the difference between using the grey and the green banner? 

## Data Contamination

You show this analysis to your colleague. She raises your concern that some participants in your data set may not be internalizing the social messaging information. Perhaps they are just clicking through the experiment. 

Fortunately, you have recorded how much time participants spend on the experimenet. Let's visualize that data.

~~~~
viz.hist(_.pluck(bannerData, "time"), {numBins: 10})
~~~~

This look likes canonical wait time data, following a log-normal distribution. To validate this intuition, let's look at the data by taking the log.

~~~~
var logTimeData = map(function(t){
	return Math.log(t);
}, _.pluck(bannerData, "time"))

viz.hist(logTimeData, {numBins: 10})
~~~~

Looks pretty normal, but also looks like there's something funny going on. Some people are spending substantially less time on your experiment than other people. Your colleague might be right: Presumably, none of these people clicking through the experiment are going to be internalizing the subtle social messaging. 

How can we account for this potential data contamination? We posit 2 groups of visitors: `"bonafide"` participants and `"disingenuous"` participants, and they plausibly have different rates of donating.

~~~~
///fold:
var foreach = function(lst, fn) {
    var foreach_ = function(i) {
        if (i < lst.length) {
            fn(lst[i]);
            foreach_(i + 1);
        }
    };
    foreach_(0);
};
///

var model = function() {

  // average time spent on the website (in log-seconds)
  // assume the disingenuous participants spend less time on the experiment (because they click through)
  var logTimes = {
    bonafide: gaussian(3,3), // exp(3) ~ 20s
    disingenuous: gaussian(0,2), // exp(2) ~ 7s
  }

  // variance of time spent on website (plausibly different for the two groups)
  var sigmas =  {
    bonafide: uniform(0,3),
    disingenuous: uniform(0,3),
  }

  var donationRates = {
    experimental: uniform(0,1),
    control: uniform(0,1)
  };

  // mixture parameter (i.e., % of bonafide visitors)
  var probBonafide = uniform(0,1);

  foreach(bannerData, function(personData) {

      var group = flip(probBonafide) ? "bonafide" : "disingenuous";

      observe(
      	Gaussian({mu: logTimes[group], sigma: sigmas[group]}), 
      	Math.log(personData.time)
      	)

      // disingenuous visitors have a very low probability of donating
      var acceptanceRate = (group == "bonafide") ? 
      	donationRates[personData.condition] : 
      	0.0000001

      observe(Bernoulli({p:acceptanceRate}), personData.converted)

  } )

  return { logTimes_disingenuous: logTimes.disingenuous,
            logTimes_bonafide: logTimes.bonafide,
            sigma_disingenuous: sigmas.disingenuous,
            sigma_bonafide: sigmas.bonafide,
            experimental: donationRates.experimental,
            control: donationRates.control,
            percent_bonafide: probBonafide }

}

var numSamples = 100000;
var posterior = Infer({method: "incrementalMH", 
                       samples: numSamples, burn: numSamples/2,
                   		verbose: true, verboseLag: numSamples/10}, 
                      model)

// run a big model: takes about 1 minute
editor.put("posterior", posterior)
~~~~

#### Examine posterior

Display marginal posterior over the rate of bonafide participants.

~~~~
///fold:
var marginalize = function(myDist, label){
    Infer({method: "enumerate"}, function(){
        var x = sample(myDist);
        return x[label]
    });
};
///

var jointPosterior = editor.get("posterior");

var marginalBonafide = marginalize(jointPosterior, "percent_bonafide");

viz.hist(marginalBonafide, {numBins: 15});
~~~~
	
So, indeed, almost 15% of your participants were inferred to be "disingenuous".

~~~~
///fold:
var marginalizeExponentiate = function(myDist, label){
    Infer({method: "enumerate"}, function(){
        var x = sample(myDist);
        return Math.exp(x[label])
    });
};
///

var jointPosterior = editor.get("posterior");

var marginalTime_accidental = marginalizeExponentiate(jointPosterior, "logTimes_disingenuous");
var marginalTime_bonafide = marginalizeExponentiate(jointPosterior, "logTimes_bonafide");

print("Inferred time spent by accidental visitors (in seconds)")
viz.hist(marginalTime_disingenuous, {numBins: 10});

print("Inferred time spent by bonafide visitors (in seconds)")
viz.hist(marginalTime_bonafide, {numBins: 10})
~~~~

### Exercises

1. Return the marginal distributions over the rates of the conversion parameters. (Use the [`marginalize()`](http://docs.webppl.org/en/master/functions/other.html#marginalize) function). Does accounting for the accidental visitors change the conclusions you can draw about the efficacy of the experimental condition?

2. You show these results to your friend, and she is surprised by them. Why are the results the way they are? In the above model, we assumed the rate of bonafide visitors was independent of which condition they were assigned. Could that be incorrect? Modify the model above to test the hypothesis that the rate of bonafide visitors was different for the different banners.

3. In the above model, we assume that disingenuous visitors are very unlikely to donate. How could we relax this assumption, and say that disingenuous visitors also have some probability of "accidentally" donating their money? Modify the model to express this possibility, run the model, and draw inferences about the rate at which disingenuous visitors buy your product.

## Different, or multiple, dependent measures

So far, we have been examining the hypothesis of whether the an experimental social messaging influences participants' propensity to donate their compensation to a social cause. Your colleague reminds you that you also collected ratings about their belief in the importance of the cause. Those ratings were given on a 1-to-5 scale, where 5 was a strong endorsement of the cause and a 1 was a strong endorsement of the opposite cause.

We could analyze these data separately, and see if there was any effect on the endorsement data. But insofar as we believe these two measurements are the byproduct of the same latent construct (e.g., the underlying support for the cause), we should be able to articulate a way in which they are related.


~~~~
///fold:
var foreach = function(lst, fn) {
    var foreach_ = function(i) {
        if (i < lst.length) {
            fn(lst[i]);
            foreach_(i + 1);
        }
    };
    foreach_(0);
};

var levels = function(df, label){
  return _.uniq(_.map(df, label));
}
///


// list of unique conditions (banners)
var conditions = levels(bannerData, "condition");

// list of unique browsers
var browsers = levels(bannerData, "browser");

var model = function() {

  // mixture parameter (i.e., % of bonafide visitors)
  var probBonafide = uniform(0,1);


  // average time spent on the website (in log-seconds)
  // bonafide visitors are those that stay on average more than just a few seconds
  var logTimes = {
    bonafide: gaussian(3,3), // exp(3) ~ 20s
    accidental: gaussian(0,2), // exp(2) ~ 7s
  }

  // variance of time spent on website (plausibly different for the two groups)
  var sigmas =  {
    bonafide: uniform(0,3),
    accidental: uniform(0,3),
  }

  var conversionRates = {
    green: uniform(0,1),
    grey: uniform(0,1)
  };


  map(function(browser){
  	var dataForOneBrowser = _.filter(bannerData, {browser: browser});


  }, browsers)

  bannerData)

  foreach(bannerData, function(personData) {

      var group = flip(probBonafide) ? "bonafide" : "accidental";

      observe(
      	Gaussian({mu: logTimes[group], sigma: sigmas[group]}), 
      	Math.log(personData.time)
      	)

      // accidental visitors have a very low probability of buying your product
      var acceptanceRate = (group == "bonafide") ? 
      	conversionRates[personData.condition] : 
      	0.0000001

      observe(
      	Bernoulli({p:acceptanceRate}), 
      	personData.converted
      	)

  } )

  return { logTimes_accidental: logTimes.accidental,
            logTimes_bonafide: logTimes.bonafide,
            sigma_accidental: sigmas.accidental,
            sigma_bonafide: sigmas.bonafide,
            green: conversionRates.green,
            grey: conversionRates.grey,
            percent_bonafide: probBonafide }

}

var numSamples = 100000;
var posterior = Infer({method: "incrementalMH", 
                       samples: numSamples, burn: numSamples/2,
                   		verbose: true, verboseLag: numSamples/10}, 
                      model)

// run a big model: takes about 1 minute
editor.put("posterior", posterior)
~~~~


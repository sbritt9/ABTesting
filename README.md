A/B Testing
================

#### Author: Stephen Britton

#### Date: Feb 2018

Experiment Design
=================

At the time this experiment was running, Udacity had two options on the course overview:

-   Start Free Trial
-   Access Course Materials

When a user would click on Start Free Trial, they are asked to enter credit card information then enrolled in a free trial for the paid version of the course. After 14 days, they will be charged if they have not yet canceled their trial.

If the user clicked on Access Course Materials, they will be able to view the videos and take quezzes for free, but they will not receive coaching, recieve coaching or be able to submit their final project for feedback.

For the experiment, Udacity wanted to test a change where if the user clicked "start free trial", they were asked how much time they had available to devote to the course. If the user indicated 5 or more hours per week, they would be taken through to the checkout process. If they indicated fewer than 5 hours, a message would appear indicating that Udacity courses require a greater time commitment and suggested that the user may instead access the course materials for free. At that point the user would have the option to continue enrolling in the free trial, or access materials for free.

The hypothesis was that if they set clearer expectations up front, they could reduce the amount of frustrated users who left the free trial because they didn't have enough time to commit - without significantly reducing the number of users to continue past the free trial and eventually complete the course.

If the hypothesis was true, Udacity could improve the overall user experience for those who are likely to complete the course. The unit of diversion is a cookie, although if the user enrolls in the free trial, they are then tracked by unique user-id.

Metric Choice
-------------

### Invariant Metrics

-   Number of cookies: number of unique cookies to view the course overview.
    -   This is an invariant metric because we don't expect to see a drastic change between the control and experiment groups based on the change being conducted.
-   Number of clicks: Number of unique cookies to click the "Start Free Trial" button
    -   This is also an invariant metric because like number of cookies, this is unexpected to change between control and experiment groups. This change is not targeting a higher rate of clicks.
-   Click-through-probability: Number of unique cookies to click the "Start Free Trial" button divided by the number of unique cookies to view the course overview page.
    -   This is an invariant metric because like cookies and clicks, the change is not directed at increasing this metric. It is unexpected to change between control and experiment groups.

    It is noted that all of the invariant metrics are collected before the scope of the experiment begins.

### Evaluation Metrics

-   Gross Conversion: number of users who complete checkout and enroll in a free trial divided by the number of unique clicks
    -   Because the goal of the experiment aims to reduce the amount of users who leave a free trial because they do not have enough time to complete the course, gross conversion is a good way to measure if there is a reduction in users who start the free trial who may not have time to dedicate.
-   Net Conversion: number of users who remain enrolled in the free trial long enough to convert to a paid user divided by the number of unique clicks
    -   This metric serves as a gauge of users who remain enrolled after being introduced to the time committment needed for a course. The hypothesis would ultimately aim to reduce the amount of sign ups without reducing the amount of users who continue past the free trial and complete the course.

### Unused Metrics

-   Number of user-ids: Number of users who enroll in the free trial
    -   A potential backup evaluation metric that could have been used as a measure of success; however, the two evaluation metrics chosen were a better fit of success.
-   Retention: Number of users that begin a free trial then convert into a paid account
    -   This metric relies on different unit of diversion and unit of analysis which can result in inaccuracies in analysis proving to be an ineffective evaluation metric and is subject to change disqualifying it from being an invariant metric.

Measuring Standard Deviation
----------------------------

Assuming a sample size of 5000 and a binomial distribution, the standard deviation will be calculated with the following formula:

√((p(1-p))/n)

Yielding the following values for our two evaluation metrics:

-   Gross Conversion Standard Deviation: 0.0202
-   Net Conversion Standard Deviation: 0.0156

The number of cookies serves as the unit of diversion for our experiment and similarly, with both gross conversion as well as net conversion the unit of analysis is also number of cookies. Because they are both reliant on the same metric, we can assume that the analytical estimate is comparable to the empirical variability and there is not significant need to calculate an empirical estimate.

Sizing
------

### Number of Samples vs. Power

Our two evaluation metrics: gross conversion and net conversion are highly correlated. Because of this, we will not need to use the Bonferonni Correction during the analysis phase.

The relevant values needed for calculating sample size and page views are as follows:

-   Gross Conversion
    -   Baseline (final project baseline values): 0.20625
    -   α: 0.05
    -   1−β: 0.8 (1 - 0.2)
    -   dmin: 0.01 (minimum detectable effect)
-   Net Conversion
    -   Baseline (final project baseline values): 0.20625
    -   α: 0.05
    -   1−β: 0.8 (1 - 0.2)
    -   dmin: 0.0075 (minimum detectable effect)

Using the online calculator - "Evan's Awesome A/B Tools" we can calculate that the sample size we need to power our experiment is the following:

Gross Conversion: 25,835

Net Conversion: 27,413

Given our sample size, we can use the larger of the two to determine the amount of pageviews required for the experiment. Because we need enough pageviews for both the control as well as the experiment group, we will calculate pageview requirement as follows:

Net Conversion Sample Size / Click-through-probability \* 2

27,413 / 0.08 \* 2 = 685,325

The number of pageviews needed to power the experiment is: **685,325**

### Duration vs. Exposure

According to unique cookies collected per day at 40,000, we can calculate the number of days it will take to achieve the number of required pageviews. We will also factor in the amount of traffic we would like to divert between them to adjust how long the experiment will take. Ideally, this should only take a few weeks or it will not be a reasonably experiment.

Target Pageviews / Cookies \* Proposed Diversion = Duration

To keep this experiment within a reasonable timeframe of just over 3 weeks, we will divert 75% of the traffic giving us a duration of **23**.

685325 / (40,000 \* .75) = 22.8 (rounded up to 23)

While 75% seems high, this experiment does not provide medical advice and does not involve sensitive information. Because it has a relatively low risk, keeping the schedule at a minimum can be a priority.

Experiment Analysis
===================

Sanity Checks
-------------

Using the provided data, we can calculate the values we expect to observe within a 95% confidence interval. We will check our three invariant metrics as follows:

``` r
# Read data from CSV
control <- read.csv("control.csv", header = TRUE, check.names = FALSE, na.strings = c(""))

experiment <- read.csv("experiment.csv", header = TRUE, check.names = FALSE, na.strings = c(""))

# Invariant Metrics to check:
# Number of Cookies: cookies
# Number of Clicks: clicks
# Click Through Probability: ctp

# variables
prob <- .5
zscore <- 1.96

# functions
standardError <- function(p, n){ return(sqrt((p*(1-p))/(n))) }
marginOfError <- function(x){ return(zscore * x) }
lowerBound <- function(x, y){ return(round(y - x, digits=4)) }
upperBound <- function(x, y){ return(round(y + x, digits=4)) }
isPassing <- function(prob, lower, upper)
  { 
  return (prob >= lower && prob <= upper) 
}
output <- function(subject, lower, upper, observe, passes){
  cat(subject, '\n',
    'Lower Bound:', lower, '\n',
    'Upper Bound:', upper, '\n',
    'Observed:  ', observe, '\n',
    'Passes:', passes, '\n\n')
}

# Generate values for number of cookies analysis

# Totals
control.views <- sum(control$Pageviews)
experiment.views <- sum(experiment$Pageviews)
total.views <- control.views + experiment.views

# Values
cookies.obs <- round(control.views / total.views, digits=4)
cookies.se <- standardError(prob, total.views)
cookies.margin <- marginOfError(cookies.se)
cookies.lower <- lowerBound(cookies.margin, prob)
cookies.upper <- upperBound(cookies.margin, prob)
cookies.pass <- isPassing(cookies.obs, cookies.lower, cookies.upper)

# Generate values for number of clicks analysis

# Totals
control.clicks <- sum(control$Clicks)
experiment.clicks <- sum(experiment$Clicks)
total.clicks <- control.clicks + experiment.clicks

# Values
click.obs <- round(control.clicks / total.clicks, digits=4)
click.se <- standardError(prob, total.clicks)
click.margin <- marginOfError(click.se)
click.lower <- lowerBound(click.margin, prob)
click.upper <- upperBound(click.margin, prob)
click.pass <- isPassing(click.obs, click.lower, click.upper)

# Generate values for Click-Through-Probability

control.ctp <- control.clicks / control.views
experiment.ctp <- experiment.clicks / experiment.views
ctp.pool <- round((control.ctp + experiment.ctp) / 2, digits=4)
ctp.difference <- experiment.ctp - control.ctp
ctp.se <- standardError(ctp.pool, control.views)
ctp.margin <- marginOfError(ctp.se)
ctp.lower <- lowerBound(ctp.margin, control.ctp)
ctp.upper <- upperBound(ctp.margin, control.ctp)
ctp.pass <- isPassing(ctp.pool, ctp.lower, ctp.upper)

output("Number of Cookies", cookies.lower, cookies.upper, cookies.obs, cookies.pass)
```

    ## Number of Cookies 
    ##  Lower Bound: 0.4988 
    ##  Upper Bound: 0.5012 
    ##  Observed:   0.5006 
    ##  Passes: TRUE

``` r
output("Number of Clicks", click.lower, click.upper, click.obs, click.pass)
```

    ## Number of Clicks 
    ##  Lower Bound: 0.4959 
    ##  Upper Bound: 0.5041 
    ##  Observed:   0.5005 
    ##  Passes: TRUE

``` r
output("Click Through Probability", ctp.lower, ctp.upper, ctp.pool, ctp.pass)
```

    ## Click Through Probability 
    ##  Lower Bound: 0.0812 
    ##  Upper Bound: 0.083 
    ##  Observed:   0.0822 
    ##  Passes: TRUE

All three invariant metrics pass the sanity check and do not require further testing.

Result Analysis
---------------

### Effect Size Tests

For each of your evaluation metrics, give a 95% confidence interval around the difference between the experiment and control groups. Indicate whether each metric is statistically and practically significant.

``` r
# Analysis runs on non-null rows only - conveniently the first 23

control.clicks <- sum(control$Clicks[1:23])
experiment.clicks <- sum(experiment$Clicks[1:23])
total.clicks <- control.clicks + experiment.clicks

# Calculate Enrollment Metrics
control.enroll <- sum(control$Enrollments[1:23])
experiment.enroll <- sum(experiment$Enrollments[1:23])
total.enroll <- control.enroll + experiment.enroll


# Calculate Payment Metrics
control.pay <- sum(control$Payments[1:23])
experiment.pay <- sum(experiment$Payments[1:23])
total.pay <- control.pay + experiment.pay

# Functions

isStatistical <- function(lower, upper){
  containsZero <- lower <= 0 & upper >= 0
  if(containsZero){
    return("No")
  }
  return("Yes")
}

isPractical <- function(dmin, lower, upper){
  contains_dMin <- (lower <= dmin & upper >= dmin) | (lower <= -dmin & upper >= -dmin)
  if(contains_dMin){
    return("No")
  }
  return("Yes")
}

outputAnalysis <- function(subject, lower, upper, difference, isStatSig, isPracSig){
    cat(subject, '\n',
    'Lower Bound:', lower, '\n',
    'Upper Bound:', upper, '\n',
    'Difference:  ', difference, '\n',
    'Statistically Significant:', isStatSig, '\n',
    'Practically Significant:', isPracSig, '\n\n')
}
```

Gross Conversion:

``` r
# Gross Conversion dmin = 0.01

gross.pool <- total.enroll / total.clicks
gross.difference <- (experiment.enroll/experiment.clicks) - (control.enroll/control.clicks)
gross.se <- sqrt((gross.pool*(1-gross.pool))*((1/control.clicks)+(1/experiment.clicks)))
gross.margin <- marginOfError(gross.se)
gross.lower <- lowerBound(gross.margin, gross.difference)
gross.upper <- upperBound(gross.margin, gross.difference)
gross.statistical <- isStatistical(gross.lower, gross.upper)
gross.practical <- isPractical(0.01, gross.lower, gross.upper)

outputAnalysis("Gross Conversion Analysis",
               gross.lower,
               gross.upper,
               gross.difference,
               gross.statistical,
               gross.practical)
```

    ## Gross Conversion Analysis 
    ##  Lower Bound: -0.0291 
    ##  Upper Bound: -0.012 
    ##  Difference:   -0.02055487 
    ##  Statistically Significant: Yes 
    ##  Practically Significant: Yes

Gross Conversion *does not* include an interval of 0 and does not cross the practical significance boundary (dmin = 0.01); therefore gross conversion is both practically and statistically signifigant.

Net Conversion:

``` r
# Net Conversion dmin = 0.0075

net.pool <- total.pay / total.clicks
net.difference <- (experiment.pay/experiment.clicks) - (control.pay/control.clicks)
net.se <- sqrt((net.pool*(1-net.pool))*((1/control.clicks)+(1/experiment.clicks)))
net.margin <- marginOfError(net.se)
net.lower <- lowerBound(net.margin, net.difference)
net.upper <- upperBound(net.margin, net.difference)
net.statistical <- isStatistical(net.lower, net.upper)
net.practical <- isPractical(0.0075, net.lower, net.upper)

outputAnalysis("Net Conversion Analysis",
               net.lower,
               net.upper,
               net.difference,
               net.statistical,
               net.practical)
```

    ## Net Conversion Analysis 
    ##  Lower Bound: -0.0116 
    ##  Upper Bound: 0.0019 
    ##  Difference:   -0.004873723 
    ##  Statistically Significant: No 
    ##  Practically Significant: No

Net Conversion *does* include 0 in the confidence interval and crosses the practical significance boundary (dmin = 0.0075); therefor net conversion is not practically or statistically significant.

### Sign Tests

For each of your evaluation metrics, do a sign test using the day-by-day data, and report the p-value of the sign test and whether the result is statistically significant.

Two-tailed t-tests calculated using Graphpad's "QuickCalcs" testing calculator: <!-- https://www.graphpad.com/quickcalcs/binomial1.cfm -->

-   Gross Conversion:
    -   Number of observed "successes": 4
    -   Number of trials/experiments: 23
    -   Probability: .5
    -   p-value: .0026
    -   Statistically Significant?: Yes
-   Net Conversion:
    -   Number of observed "successes": 10
    -   Number of trials/experiments: 23
    -   Probability: .5
    -   p-value: .6776
    -   Statistically Significant?: No

### Summary

This experiment was run to determine whether or not additional prompts should be added when a user clicks on "Start Free Trial" inquiring about the available time the user had to dedicate to the coursework.

Based on the evaluation metrics we chose: Gross Conversion and Net Conversion we calculated a number of values that would assist us in determining whether or not to make this change. Because these two metrics are correlated, we did not use Bonferroni correction as this would have impacted the statistical error and may have introduced false negatives.

The analysis shows that gross conversion is practically and statistically significant; however net conversion is neither practically or statistically significant. Both sign and effect tests showed the same findings so there are no discrepencies to be observed.

Recommendation
--------------

Based on the design goals of the experiment, we can see that there was a reduction of user that enrolled based on the prompt - as seen in the gross conversion metric analysis. This impact was statistically as well as practically significant; however, there was no significant changes with net conversion and the confidence interval exceeded the negative practical significance boundary. Given that a reduction of sign ups was not matched with a reduction of conversions, we can conclude that we should not recommend this change to be launched.

Follow-Up Experiment
====================

To reduce the quantity of frustrated users who abandon courses, it would be probable to measure daily sign-in activities that could result in extension of the free trial on conclusion of 7 and 14 days.

This feature inclusion could encourage return use and decrease abandonment leading to possible increase in net conversion.

Hypothesis

I would estimate that including this feature would reduce the amount of users who abandon a course after starting a free trial, potentially increasing user participation and net conversions.

Invariant metric

User-ids: this metric would remain the same as user-id's are not deleted when a user abandons a course. This would also suggest that the user has already enrolled in a free trial and has already created a sign in id.

Evaluation metrics

Payment Conversion: this metric captures the amount of users who continue past the free trial and make a payment to stay enrolled in the course. This is an effective metric to test whether or not being actively involved with daily sign-on would lead to increased satisfaction and conversion to paid accounts.

Course Abandonment: this metric would measure the amount of users who started with a course then abandoned the course after a certain amount of days. Abandonment of a course would drastically reduce the probability a user would pay for the course as they are no longer actively involved.

Unit of Diversion

Unit of diversion would be course abandonment, this metric would allow me to gather data on user participation. This is different from the unit of analysis which would be used to pitch a conclusion rather than data collection.

References
==========

-   GraphPad QuickCalcs: <https://www.graphpad.com/quickcalcs/binomial2/>
-   Evan's Awesome A/B Tools: <http://www.evanmiller.org/ab-testing/sample-size.html>
-   Research Design Matters: <http://re-design.dimiter.eu/?p=253>
-   A/B Testing: The most powerful way to turn clicks into customers
    -   Author: Dan Siroker & Pete Koomen
    -   Published: Aug 7, 2013
    -   Publisher: Wiley
-   Small Table of z-values for Confidence Intervals: <http://www.ltcconline.net/greenl/Courses/201/Estimation/smallConfLevelTable.htm>
-   Markdown Cheatsheet: <http://assemble.io/docs/Cheatsheet-Markdown.html>

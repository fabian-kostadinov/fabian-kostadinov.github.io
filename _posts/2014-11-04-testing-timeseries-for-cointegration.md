---
layout: post
title: Testing Time Series for Cointegration
tags: [statistics, timeseries, cointegration]
---
Cointegration is an important concept when dealing with time series data. Here's the corresponding [definition on Wikipedia](http://en.wikipedia.org/wiki/Cointegration):

<blockquote>Cointegration is a statistical property of time series variables. Two or more time series are cointegrated if they share a common stochastic drift.</blockquote><!--more-->

In other (rather non-scientific) words, if both time series are non-stationary _and_ they share a trend together (which can be explained through the existence of a common cause), then they are _cointegrated_.

Cointegration is not the same as correlation!
<iframe width="560" height="315" src="//www.youtube.com/embed/vrryb49jbIo" frameborder="0" allowfullscreen></iframe>

<div class="message">
<em>Correlation</em> measures the co-movement between two time series, it answers the question: How much do they move together? It does not guarantee that the two measures stay close to each other in the long-run.<br/>
<br/>
<em>Cointegration</em> means that two time series will not deviate substantially from each other, yet when they do, the gap will be closed sooner or later again.</div>

For this reason, it is certainly possible for two time series to be correlated but not cointegrated, cointegrated but not correlated, both or none.

Cointegration is an often encountered feature of economic or financial time series. A typical text book example is a country's consumption and income. The more people earn, the more they have left to consume - of course assuming stable prices. Both time series usually grow over time. We would therefore assume consumption and income to be cointegrated time series.

Or consider an investor who wants to build a portfolio. For cointegrated stocks, a significant deviation between the two stocks will soon close again. An example would probably be gold and silver prices. If the two deviate significantly from each other, an arbitrage opportunity exists. (Someone could buy the relatively cheaper metal and sell the relatively more expensive metal, waiting for the gap to close again.) There are some really interesting articles out there on this topic, see the references section.

What we need is a statistical test for cointegration. There are different such tests, but the most common one is probably the [Augmented Dickey-Fuller (ADF) test](http://en.wikipedia.org/wiki/Augmented_Dickey%E2%80%93Fuller_test). The ADF test returns a negative value. The more negative this value is, the higher the probability that the null hypothesis - "There is no cointegration present in the compared time series." - can be rejected. Whereas the ADF test is available for nearly all statistics software, unfortunately there is no simple Excel formula for it. (There is however an [AddIn provided by Kurt Annen](http://www.web-reg.de/adf_addin.html).)

For the statistics software R, there is a great introductory article written by [Paul Teetor](http://quanttrader.info/public/) available at [http://quanttrader.info/public/testForCoint.html](http://quanttrader.info/public/testForCoint.html). Besides explaining how to calculate an ADF test, it also shows all the steps how to import your data into R from a CSV file and how to prepare it for analysis.

This involves three steps.

First, we calculate a measure for the "co-movement" of both series. For this purpose, we use a simple linear regression formula between the two time series. It does not really matter which one is selected as the "dependent" and which one as the "independent" series, because we do not claim that there exists a "dependency relation" between the two. Be aware that we are not interested in the intercept, but only in the _beta (&beta;)_, that is the regression coefficient. This _beta_ tells us something about how strongly a change in one time series is accompanied by a corresponding change in the other time series. Therefore, we want to solve the following regression formula:

<code>X<sub>time series 1</sub> = (-&beta;) * X<sub>time series 2</sub></code>

In R, we can use the <code>lm</code> function to solve this regression formula, in Excel 2013 we can perform a regression analysis (under Data -> Data Analysis -> Regression).

Second, we can calculate a new time series of "spreads" or "differences" between values of the two original time series using the formula:

<code>spread<sub>t</sub> = x<sub>t, time series 1</sub> - &beta; * x<sub>t, time series 2</sub></code>

Third, we apply the ADF test on the new time series of spreads. Our null hypothesis is: "The spread time series is not-stationary." If we can reject this null hypothesis at a, let's say, 95% level, then we can accept the alternative hypothesis: "The spread time series is indeed stationary." In R, there is for instance the <code>adf.test</code> function.

<code>adf.test(spreads, alternative="stationary", k=0)</code>

The function returns a _Dickey Fuller_ statistical value, and, thankfully, also a probability value _p-value_ which can be interpreted more easily. If the _p-value_ is < 0.05 (critical 5% threshold) then the spread is likely to be mean-reverting, which means that the two time series are likely to be cointegrated. Otherwise the spread is not mean-reverting, thus the two time series are unlikely to be cointegrated.

----

# References

Gekkoquant.com has some nice articles on ADF tests, cointegration, statistical arbitrage etc.:

* [http://gekkoquant.com/2012/12/17/statistical-arbitrage-testing-for-cointegration-augmented-dicky-fuller/](http://gekkoquant.com/2012/12/17/statistical-arbitrage-testing-for-cointegration-augmented-dicky-fuller/)
* [http://gekkoquant.com/2012/10/21/statistical-arbitrage-correlation-vs-cointegration/](http://gekkoquant.com/2012/10/21/statistical-arbitrage-correlation-vs-cointegration/)

Another excellent blog with various good articles on the topic is by [Ernest Chan](http://epchan.blogspot.ch). Search for "Cointegration" on the blog to find many more articles like these two:

* [http://epchan.blogspot.ch/2006/11/cointegration-is-not-same-as.html](http://epchan.blogspot.ch/2006/11/cointegration-is-not-same-as.html)
* [http://epchan.blogspot.ch/2013/11/cointegration-trading-with-log-prices.html](http://epchan.blogspot.ch/2013/11/cointegration-trading-with-log-prices.html)

Article on Pairs Trading at Godotfinance.com:

* [http://www.godotfinance.com/pdf/PairsTrading.pdf](http://www.godotfinance.com/pdf/PairsTrading.pdf)



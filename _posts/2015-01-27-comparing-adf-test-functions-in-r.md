---
layout: post
title: Comparing ADF Test Functions in R
comments: true
tags: [statistics, adf, r]
---
In one of my last posts I was not sure how R's different ADF test functions worked in detail. So, based on [this discussion thread](http://r.789695.n4.nabble.com/Howto-test-for-true-stationarity-with-adfTest-td3224977.html) I set up a simple test. I created four time series:

1. _flat0_: stationary with mean 0,
2. _flat20_: stationary with mean 20,
3. _trend0_: trend stationary with "trend mean" crossing through (0, 0) - i.e. without intercept,
4. _trend20_: trend stationary with "trend mean" crossing through (0, 20) - i.e. with intercept 20.<span class="more"></span>

{% highlight r %}
library(tseries)
library(xts)
library(fUnitRoots)

flat0 <- xts(rnorm(100), Sys.Date()-100:1)
plot(flat0)

flat20 <- xts(rnorm(100), Sys.Date()-100:1)+20
plot(flat20)

trend0 <- flat0+(row(flat0)*0.1)
plot(trend0)

trend20 <- flat0+(row(flat0)*0.1)+20
plot(trend20)
{% endhighlight %}
Here are corresponding plots:
![flat0](/public/img/2015-01-27-comparing-adf-functions-in-r-flat0.png "flat0")
Notice the mean at the zero line and the absence of a trend.

![flat0](/public/img/2015-01-27-comparing-adf-functions-in-r-flat20.png "flat20")
Notice the mean at the 20 line and the absence of a trend.

![flat0](/public/img/2015-01-27-comparing-adf-functions-in-r-trend0.png "trend0")
Notice that the trending mean crosses the origin (0, 0) and the presence of a trend with slope 0.1.

![flat0](/public/img/2015-01-27-comparing-adf-functions-in-r-trend20.png "trend20")
Notice that the trendin mean crosses the (20, 0) point and the presence of a trend with slope 0.1

Next, I calculated ADF tests using both the <code>adf.test</code> function in the _tseries_ package and the <code>adfTest</code> function in the _fUnitRoots_ package. We must keep in mind the following few points:

* <code>adf.test</code> in _tseries_ always automatically detrends the given time series.
* <code>adfTest</code> in _fUnitRoots_ has three different type options: <code>nc</code>, <code>c</code> and <code>ct</code>.

From R's documentation of the <code>adfTest</code> function:
<blockquote>_type_: a character string describing the type of the unit root regression. Valid choices are "nc" for a regression with no intercept (constant) nor time trend, and "c" for a regression with an intercept (constant) but no time trend, "ct" for a regression with an intercept (constant) and a time trend. The default is "c".</blockquote>

So, let's run the tests.

{% highlight r %}
adf.test(flat0, alternative = "stationary", k = 0)
adf.test(flat20, alternative = "stationary", k = 0)
adf.test(trend0, alternative = "stationary", k = 0)
adf.test(trend20, alternative = "stationary", k = 0)

adfTest(flat0, lags = 0, type = "nc")
adfTest(flat20, lags = 0, type = "nc")
adfTest(trend0, lags = 0, type = "nc")
adfTest(trend20, lags = 0, type = "nc")

adfTest(flat0, lags = 0, type = "c")
adfTest(flat20, lags = 0, type = "c")
adfTest(trend0, lags = 0, type = "c")
adfTest(trend20, lags = 0, type = "c")

adfTest(flat0, lags = 0, type = "ct")
adfTest(flat20, lags = 0, type = "ct")
adfTest(trend0, lags = 0, type = "ct")
adfTest(trend20, lags = 0, type = "ct")
{% endhighlight %}

The following table contains the p-values produced by the corresponding ADF function. If the p-value is lower than 0.05 it is significant at the 95% confidence threshold.

<table>
  <tr>
    <th>ADF function</th>
    <th>flat0</th>
    <th>flat20</th>
    <th>trend0</th>
    <th>trend20</th>
  </tr>
  <tr>
    <td><code>adf.test(&lt;series&gt;, alternative = "stationary", k = 0</code></td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
  </tr>
  <tr>
    <td><code>adfTest(&lt;series&gt;, lags = 0, type = "nc")</code></td>
    <td>&lt; 0.01</td>
    <td>0.5294</td>
    <td>0.4274</td>
    <td>0.7736</td>
  </tr>
  <tr>
    <td><code>adfTest(&lt;series&gt;, lags = 0, type = "c")</code></td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
    <td>0.1136</td>
    <td>0.1136</td>
  </tr>
  <tr>
    <td><code>adfTest(&lt;series&gt;, lags = 0, type = "ct")</code></td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
    <td>&lt; 0.01</td>
  </tr>
</table>
Voil&agrave;! We can now understand much better what the different functions and type parameters effectively do.

The first function <code>adf.test</code> always indicates a significant stationarity - even if the time series are clearly trending. This is of course because it first detrends the time series, and thus the returned p-value does not distinguish between true stationarity and trend stationarity. Neither does it seem to make any difference concerning the "absolute level", i.e. the intercept at which the time series is.

The fourth function <code>adfTest(&lt;series&gt;, lags = 0, type = "ct")</code> returns the same results as <code>adf.test</code>. In other words, it both detrends and handles an occurring intercept. This is not suited to distinguish between true stationarity and trend stationarity.

The second function <code>adfTest(&lt;series&gt;, lags = 0, type = "nc")</code> clearly distinguishes between each case: intercept vs. no intercept and trending vs. not trending. Only _flat0_ passes the test of being stationary around a mean of 0 and having no trend. All others have either one or the other. We must think carefully here. Should we care about spreads of two cointegrated stocks having a non-zero intercept (as long as there is no trend)? As long as we are aware that the spreads will not revert to the zero line but to another mean equal to the intercept everything is still fine. We are only interested _if_ stocks do revert to a mean, but not _where_ this mean lies. Nevertheless, at some point we of course still need to know where this mean is in order to implement our trading strategy.

The third function <code>adfTest(&lt;series&gt;, lags = 0, type = "c")</code> apparently is best suited for the purpose of identifying pairs of stocks for pair trading. No mather whether our calculated spreads are centered around a mean (intercept) of 0 or not, it handles both situations. As soon as there is a trend however it does not any longer indicate stationarity. However, notice the relatively low p-values. Although not significant at a 95% confidence threshold, if the trend slope was less extreme (e.g. 0.05 compared to 0.1) then this test might actually ultimately pass.
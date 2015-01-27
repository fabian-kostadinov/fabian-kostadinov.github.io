---
layout: post
title: Cointegration and Total-Least-Squares Regression
comments: true
tags: [statistics, cointegration]
---
I just stumbled over a very nice [article authored by Paul Teetor on the use of total least-squares-regression in contrast to ordinary-least-squares regression for cointegration tests](http://quanttrader.info/public/betterHedgeRatios.pdf). [This blog post](http://www.cerebralmastication.com/2010/09/principal-component-analysis-pca-vs-ordinary-least-squares-ols-a-visual-explination/) also explains the same topic.

Let's quickly recall what we do when trying to find a working pairs trading strategy. First, we use one stock price time series to estimate another stock price time series. The linear regression model is <code>A = intercept + beta * B</code>, where A and B are both time series containing stock prices. From the regression estimation we retrieve the _hedge ratio_ (or _beta_ or regression _slope_) and an _intercept_. We can then compute the spreads _S_ as <code>S = A - (intercept + beta * B)</code>. Finally, we use an augmented Dickey-Fuller (ADF) test to find out whether the spreads are stationary. If they are, this implies that both stocks are in fact cointegrated. Gaps between the two stock prices will not persist infinitely long, but close after some time.

In R you can use the <code>lm</code> function to create a linear regression estimation: <code>lm(StockA ~ StockB)</code>, or <code>lm(StockA ~ StockB + 0)</code> if you want to explicitly set the intercept to 0. The _lm_ function however relies on an ordinary-least-squares regression method, and both articles linked above do a nice job explaining why this is somewhat problematic. To make it short, we would like to be one hedge ratio to be the inverted value of the other, when we switch the dependent and the independent variable. However, this is not the case with ordinary-least-squares regression. This is from Teetor's article:
{% highlight %}
OLS for A vs. B = -0.6583409
OLS for B vs. A = -1.03481
TLS for A vs. B = -0.7611657
TLS for B vs. A = -1.313774
{% endhighlight %}
"OLS" of course means "ordinary-least-squares regression" whereas "TLS" means "total-least-squares regression". He goes on writing:
<blockquote>The OLS hedge ratios are inconsistent because 1 / -0.658 = -1.520, not -1.035, which is a substantial difference. The TLS ratios, however, are consistent because 1 / -0.761 = -1.314.
</blockquote>
The reason becomes immediately clear when you look at the pictures provided in the linked articles. In TLS regression the residuals are actually computed _orthogonal to the regression line_, whereas in ordinary-least-squares they are computed _orthogonal to the stock A_ (if A is the dependent and B the independent variable) or _orthogonal to stock B_ (if A is the independent and B the dependent variable). Using OLS regression "long A and short B" is _not_ the opposite of "long B and short A". This only becomes the case if we use TLS regression.

But how then can we calculate a total-least-squares regression in R? Unfortunately, the _lm_ function does not provide such functionality. Instead, Teetor suggests to use R's _principal component analysis_ function <code>princomp</code>.
{% highlight R %}
r <- princomp(~ StockA + StockB)
beta <- r$loadings[1, 1] / r$loadings[2,1]

# We don't really need the intercept, but it's nice to display it anyway
intercept <- r$center[2] - beta * r$center[1]

spread <- StockA - beta * StockB
{% endhighlight %}
The spreads may or may not be far away from calculating them through a OLS regression model. There are a few more details in Teetor's paper and I really recommend reading through it. In case you want to understand [how principal component analysis actually works, there is another good article written by Lindsay Smith](http://www.cs.otago.ac.nz/cosc453/student_tutorials/principal_components.pdf).
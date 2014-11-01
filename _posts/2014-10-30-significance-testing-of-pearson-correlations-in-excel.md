---
layout: post
title: Significance Testing of Pearson Correlations in Excel
comments: true
tags: [statistics, correlation, significance]
---
Yesterday, I wanted to calculate the significance of Pearson correlation coefficients between two series of data. I knew that I could use a Student's t-test for this purpose, but I did not know how to do this in Excel 2013. And, to be honest, I did not really understand the documentation of Excel's <code>T.TEST</code> formula. So, here is what I did.<!--more-->

## Pearson correlation coefficient
First, I had to calculate the corresponding Pearson correlation coefficients according to this formula:

![Pearson correlation coefficient formula](/public/img/20141030-pearson-correlation-coefficient-formula.png "Pearson correlation coefficient formula")

where _r<sub>xy</sub>_ is the Pearson correlation coefficient, _n_ the number of observations in one data series, _<span style="text-decoration: overline;">x</span>_ the arithmetic mean of all _x<sub>i</sub>_, _<span style="text-decoration: overline;">y</span>_ the arithmetic mean of all _y<sub>i</sub>_, _s<sub>x</sub>_ the standard deviation for all _x<sub>i</sub>_, and s<sub>y</sub> the standard deviation for all _y<sub>i</sub>_.

Let's assume, the data series to be correlated are stored in arrays <code>A1:A100</code> and <code>B1:B100</code>, thus _n = 100_:

<code>=PEARSON(A1:A100;B1:B100)</code>

Alternatively, you could also use the Correl function, which returns the same result:

<code>=CORREL(A1:A100;B1:B100)</code>

(I am using a Swiss German localization, therefore Excel's delimiter for formula arguments is a semicolon <code>;</code> rather than a comma <code>,</code> in my case.)

Naturally, the returned correlation value is in the range of -1.0 to +1.0. This value is often referred to as _Pearson r_ or r<sub>xy</sub> in our case.

## t-value
Next, I calculated the corresponding t-values according to this formula:

![t-value formula](/public/img/20141030-t-value-formula.png "t-value formula")

where _t_ is the t-value, which can be positive for positive correlations or negative for negative correlations, _r<sub>xy</sub>_ is the already calculated Pearson correlation coefficient, and _n_ is the number of observations again (here _n = 100_).

## Significance testing
Finally, I needed to decide whether the computed t-values were actually significant or not. For this purpose, we need to compare them to pre-calculated t-values available in a [t-value table](http://en.wikipedia.org/wiki/Student%27s_t-distribution#Table_of_selected_values). To do so, it is necessary to decide upon two things:

1. What is the desired significance or confidence level, e.g. 95% or 99%?
2. Do I want to use a one-tailed or a two-tailed t-test?

In my case, I decided to use a 95% significance level (which is a very common choice for data that are not highly critical). What confused me for some time is that some sources like the already linked [Wikipedia-article](http://en.wikipedia.org/wiki/Student%27s_t-distribution#Table_of_selected_values) express confidence __positively__ by stating the degree of certainty (e.g. 0.95 or 95%), whereas others [like this one](http://www.socr.ucla.edu/applets.dir/t-table.html) express confidence __negatively__ by the probability for being wrong (e.g. 0.05 or 5%). Other sources again [like this one](http://www.medcalc.org/manual/t-distribution.php) actually state both in a single table. _It is essential that you know what the source you are using is referring to!_ You can always carefully compare the t-values contained in different t-tables with each other to get an understanding whether someone actually takes one or the other perspective. Just check the t-table's header line containing the probability values. Usually, the t-values are in ascending order from left to right, and in descending order from top to bottom. Hence:

<div class="message">If you want to test the significance of a positive correlation, then you must check whether your t-value <i>t</i> is greater than a certain critical positive t-value <i>t<sub>crit</sub></i> at the right side of the t-distribution: Is t > t<sub>crit</sub>?<br/>
<br/>
If you want to test the significance of a negative correlation, then you must check whether your t-value <i>t</i> is below a certain critical negative t-value <i>t<sub>crit</sub></i> at the left side of the t-distribution: Is t < t<sub>crit</sub>?</div>

Usually, a t-table only includes positive t-values, but not negative ones. How can we then test for the significance of negative correlations? It's really easy. t-distributions are symmetric with a median of 0, thus their left and right tails look exactly the same. For this reason, we can change our test from "Is t < t<sub>crit</sub>?" to "Is ABS(t) > t<sub>crit</sub>?". Instead of checking whether our t-value is below a critical negative threshold, we actually check whether its absolute value is greater than a critical positive threshold.

In case you do not have a very clear idea whether the expected correlation is actually positive or negative because it could be both it is better to use a two-tailed t-test. You can use a one-tailed t-test if you are only interested in one directionality of the correlation but not in the other (e.g. only positive but not negative, or only negative but not positive). Always remember:

<div class="message"><center><em>The probability value of a two-tailed t-value is 2x the probability value of a one-tailed t-value:</em><br/><br/>
<strong>p(t) two-tailed = 2 * p(t) one-tailed</strong></center></div>

The t-table contains in the first column the _degrees of freedom_. This is usually the number of observations _n_ (i.e. 100) minus some value depending on the context. When computing significances for Pearson correlation coefficients, this value is 2: <code>degrees of freedom = n - 2</code>.

We now have all information needed to perform the significance test. i) We have decided upon a confidence level of 95%, ii) we have decided to use a two-tailed t-test, iii) we have calculated the degrees of freedom to be 98 = 100 - 2. Looking up the critical threshold _t<sub>crit</sub>_ in the t-table we find that it is 1.984. Therefore:

* If r<sub>xy</sub> > 0 AND t > 1.984 then the Pearson correlation coefficient is significantly positive. This would be reported more compactly as: r<sub>xy</sub>(98) = &lt;value&gt;, p < 0.05 (two-tailed), where &lt;value&gt; is of course the calculated Pearson correlation coefficient.
* If r<sub>xy</sub> < 0 AND ABS(t) > 1.984 then the Pearson correlation coefficient is significantly negative.
* In all other cases the result is not significant.

---

## Using Excel for t-tests

### TINV, T.INV.2T, TDIST, and T.DIST.2T
Of course, calculating critical t-values can be done in Excel too. Before Excel 2010, there were only the TINV and TDIST formulas, now there are additionally the T.INV.2T and the T.DIST.2T formulas. All these formulas express confidence _negatively_, that is the probability value _p_ represents the probability for being wrong.

<table>
  <tr>
    <td><code>=TINV(p;df)</code></td>
    <td>returns a t-value <i>t</i> for the given probability <i>p</i> and degrees of freedom <i>df</i>, assuming a two-tailed test. <code>=TINV(p;df)</code> is equivalent to <code>=T.INV.2T(p;df)</code>.</td>
  </tr>
  <tr>
    <td><code>=T.INV.2T(p;df)</code></td>
    <td>returns a t-value <i>t</i> for the given probability <i>p</i> and degrees of freedom <i>df</i>, assuming a two-tailed test. <code>=T.INV.2T(p;df)</code> is equivalent to <code>=TINV(p;df)</code>.</td>
  </tr>
  <tr>
    <td><code>=TDIST(t;df;num-tails)</code></td>
    <td>returns a probability value <i>p</i> for the given t-value <i>t</i>, the degrees of freedom <i>df</i>, and the number of tails <i>num-tails</i> (either 1 or 2). <code>=TDIST(t;df;2)</code> is equivalent to <code>=T.DIST.2T(t;df)</code>. Furthermore, <code>=2*TDIST(t;df;1)</code> is equivalent to <code>=T.DIST.2T(t;df)</code>.</td>
  </tr>
  <tr>
    <td><code>=T.DIST.2T(t;df)</code></td>
    <td>returns a probability value <i>p</i> for the given t-value <i>t</i>, the degrees of freedom <i>df</i>, and a two-tailed t-test. <code>=T.DIST.2T(t;df)</code> is equivalent to <code>=TDIST(t;df;2)</code>.</td>
  </tr>
</table>

TINV is the inverse of the TDIST formula and vice versa.

_Examples:_ Let _p = 0.05_ (5% probability of being wrong), _n = 100_ (therefore _df = 100 - 2 = 98_). We assume a two-tailed t-test.

* Calculating _t_ for a given _p_: <code>=TINV(0.05; 100-2)</code> = <code>=T.INV.2T(0.05; 100-2)</code> = 1.9844675.
* Calculating _p_ for a given _t_: <code>=TDIST(1.9844675;100-2;2)</code> = <code>=T.DIST.2T(1.9844675;100-2)</code> = <code>=2*TDIST(1.9844675;100-2;1)</code> = 0.05.

### T.INV and T.DIST
Since Excel 2010, there is also a T.INV and a T.DIST formula. _Confusingly, they actually work quite differently from TINV and TDIST!_ First, unlike TINV and TDIST, T.INV and T.DIST by default are _one-tailed_. Second, unlike TINV and TDIST, T.INV and T.DIST actually express confidence _positively_, that is the probability value _p_ represents the degree of certainty.

<table>
  <tr>
    <td><code>=T.INV(p;df)</code></td>
    <td>returns a t-value <i>t</i> for a given probability <i>p</i> and degrees of freedom <i>df</i>, assuming a one-tailed test. <code>=T.INV(p;df)</code> is equivalent to <code>=TINV(2*(1-p);df)</code>.</td>
  </tr>
  <tr>
    <td><code>=T.DIST(p;df;true)</code></td>
    <td>returns a probability value <i>p</i> for the given t-value <i>t</i>, the degrees of freedom <i>df</i>, assuming a one-tailed t-test. <code>=2*(1-T.DIST(p;df;true))</code> is equivalent to <code>=TDIST(p;df;2)</code>. Furthermore, <code>=1-T.DIST(p;df;true)</code> is equivalent to <code>=TDIST(p;df;1)</code>.</td>
  </tr>
</table>

_Examples:_ Let _p = 0.05_ (5% probability of being wrong), _n = 100_ (therefore _df = 100 - 2 = 98_). We assume a two-tailed t-test.

* Calculating _t_ for a given _p_: <code>=T.INV(1-0.05/2;98)</code> = <code>=TINV(0.05;98)</code> = 1.9844675.
* Calculating _p_ for a given _t_: <code>=2*(1-T.DIST(1.9844675;98;true))</code> = <code>=TDIST(1.9844675;98)</code> = 0.05;
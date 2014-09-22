---
layout: post
title: Evolving Trading Strategies With Genetic Programming - Data
comments: true
---

# Part 3

Genetic programming (GP) heavily relies on existing time series data. These data are obtained from different commercial or open source data providers. Here is a list of free data providers.

* [Yahoo Finance](http://finance.yahoo.com): Provides historical daily open/high/low/close/volume (OHLCV) quotes for many stocks as comma separated value (CSV) or Excel files. Data quality does not always reach a desirable level.
* [Finanzen.ch](http://www.finanzen.ch/): Provides stock data for download similar to Yahoo Finance.
* [Dukascopy](http://www.dukascopy.com/swiss/english/marketwatch/historical/): Free FX currency pairs data. For many FX pairs data is also available at a minute bar frequency.
* [EODDATA](http://www.eoddata.com/) has options for both free and paid membership to obtain data in different formats for various trading platforms.

<small>(If you know other recommendable free data providers - especially for options data - let me know, I will add them to this list.)</small>

Many technical indicators require OHLC(V) data. Should only close data be available, these indicators cannot be computed.

Once the data is obtained, it needs to be preprocessed first. Typical tasks include:

* Changing data formats, e.g. from Excel to CSV.
* Handling missing data points, e.g. by looking them up at another data provider.
* Checking for invalid data points, i.e. violations of <code>high<sub>t</sub> &ge; open<sub>t</sub>|close<sub>t</sub> &ge; low<sub>t</sub></code>.
* Correcting for price drops of single stocks due to dividends payment. Not doing so will repeatedly unjustifiedly trigger sell rules during the training phase.
* Handling stock splits. Yahoo Finance for instance sometimes provides an additional column _Adjusted Close_, which is adjusted for stock splits and dividends.
* Handling weekends and holidays.
* Mathematical transformations. In some cases it might make sense to logarithmize input data before using it.

Because stock indices are replicated by exchange traded funds (ETFs) or derivative products such as contract for difference (CFDs), trading them actually implies trading the replica products. The replica products sometimes deviate from the stock index for one reason or another. Therefore, it is recommended to directly obtain historical data series of the replica product.

## Noise in the data

Theoretically, data obtained from different data providers should be the same for the same tradeable. This is however not necessarily the case, as this [blog entry by Daniel Fernandez](http://mechanicalforex.com/2014/09/machine-learning-in-forex-data-quality-broker-dependency-and-system-generation.html) demonstrates clearly. Applying the same trading strategy with an hourly trading frequency in the EUR/USD market on data provided by different providers the achieved results differ significanty. Fernandez observes that

1. the further back in past he goes the higher the difference of achieved results between the data sets,
2. this seems to be less of a problem if the trading frequency is lower (e.g. on a daily basis).

One of his suggestions is to deliberately introduce a certain level of random noise to the data, so that the trading strategy is able to only follow the fundamental market movements. This is repeated many times to come to a conclusion on whether the strategy is still profitable or not.

Some authors also believe that a successful trading strategy must be profitable not only for one tradeable but for several similar ones. Also, Monte Carlo simulations can be helpful to determine a strategy's historical performance.

## Lookback period

Many technical indicators have a _lookback period_. The lookback period is the number of bars that must be available to calculate the indicator. For example, a simple moving average with a window size of 10 needs at least 10 bars of data before it can be computed for the first time. In GP it might be interesting to allow technical indicators calculated on derived time series such as a moving average calculated on a "highest high of the last _n_ bars" indicator. Because the "highest high of the last _n_ bars" also requires a lookback period, the full lookback period is equal to the sum of the two lookback periods.

_Example:_

1. Indicator 1 is "highest high of last 5 bars". Starting at bar 1, the first time the indicator is available is after close of bar 5.
2. Indicator 2 is "simple moving average of last 3 bars applied on previous indicator". Starting at bar 5 the first time the indicator is available is after the close of bar 7.

In case we allow long lookback periods (such as 150 days backwards for certain slow moving averages), the aggregated lookback periods can become very long. The final length also depends on the maximum allowed rule tree depth because the deeper rules can be nested the longer the aggregated lookback periods. Should the lookback period be markedly different between two trading strategies, then the one with the shorter lookback period has an advantage over the other as it has more opportunities for trading. (This might be desired though, because it favors trading strategies with shorter lookback periods.) A solution would be to always simply taking the maximum of the available lookback period.

## Survivorship bias

A complicated issue is the [_survivorship bias_](http://en.wikipedia.org/wiki/Survivorship_bias) inherent in financial stock data. To quote [Wikipedia](http://en.wikipedia.org/wiki/Survivorship_bias):
<blockquote>In finance, survivorship bias is the tendency for failed companies to be excluded from performance studies because they no longer exist. It often causes the results of studies to skew higher because only companies which were successful enough to survive until the end of the period are included.</blockquote>
Unfortunately, stock indices like the S&P 500 or the DAX are also not free from this bias. These indices are updated regularly changing their constituents.  
At the same time some authors point out that there might also exist a _reverse survivorship bias_ in the hedge funds world, where very successful hedge funds at a point in time close their doors to the public and stop reporting further success measures.  
Finally, to complicate things even more, merger & acquisitions are common phenomena in most developed economies.

Survivorship bias is a complicated topic and difficult to account for properly. It is thus good to keep in mind that future performance might well be below historical performance.

## How much data is needed?
In statistics a general rule-of-thumb exists that at least 10x more observations or data points are needed than variable model parameters. If less data is available then the whole model building process is not really trustworthy.  
A concept closely related is the _degrees of freedom_. The degrees of freedom (df) is equal to the number of observations in the data set minus the number of model parameters that may vary independently:  
<code>df = number of data points - number of independent model parameters</code>  
Example: In case you have 5000 data points and your model has 30 independent parameters, you are left with 4970 degrees of freedom. According to [Wikipedia](http://en.wikipedia.org/wiki/Degrees_of_freedom_(statistics)#Residuals):
<blockquote>A common way to think of degrees of freedom is as the number of independent pieces of information available to estimate another piece of information.</blockquote>
A higher number of data points increases, and a higher number of model parameters decreases the degrees of freedom. The degrees of freedom are sometimes used as an input for certain statistical tests, which can be useful for building fitness functions. However, estimating the exact number of independent model parameters for a single trading strategy in GP is often more an art than a science. A simple guideline is to count the number of nodes in all sub-trees per trading strategy and possibly give complicated nodes (e.g. technical indicator nodes) a higher weight.

A fundamental problem persists though. GP tests thousands of different trading stragies with dozens of rule tree nodes on the same data set. Assuming we restrict our evolutionary process on 10 generations with 100 individuals only, each with an average individual having 15 nodes, this already results in 10 x 100 x 15 = 15'000 variable parameters being tested. Assuming we have in-sample data for 8 years of daily trading data with roughly 260 OHLCV bars per year, this results in 8 x 260 = 2080 data points, which is _much_ less than the required minimum number of data points, no matter how we count nodes. We actually needed per-minute data ensure a sufficient number of data points. But even if we had data at a per-minute frequency, we could not simply switch our trading frequency from daily to minute trading, as we would automatically enter the world of high-frequency trading, which might be quite different from a medium-term (daily) trading frequency. The only, still unsatisfactory solution left is to restrict ourselves wherever we can, that is restrict the number of generations, the population size and the average number of nodes per individual.

## In-sample vs. out-of-sample data
The obtained data is split into an in-sample (IS) period for training and an out-of-sample (OOS) control period. The IS period should contain a variety of different market situations, i.e. bullish, bearish and trendless markets. It should be the larger portion of the data. The only use of the OOS data is to ensure that there is not a significant difference between the behavior of the trading strategy in the IS and the OOS data. Because both IS and OOS data are both historical data, a new trading strategy must first be tested in a walk-forward setting on real-market data for a certain time period. Only if the strategy's performance continuously persists should it be considered trustworthy.

## Multiple tests bias
Rarely is the evolutionary process run only once. A far more common work procedure is to let it run once, look at the IS and OOS results, change some settings and run the evolutionary process again. This work procedure is inherently dangerous as it carries a _multiple tests bias_. The more the GP process is run, the higher the chance that an apparently promising trading strategy is finally found both according to the IS and OOS performance. In statistics it is common to use confidence intervals to express the degree of confidence: "This trading strategy is profitable at a 95% confidence interval." In other words, there is a 1 in 20 chance that this strategy only looks profitable but in actuality is not. Another interpretation of a confidence level at 95% is that out of 20 tested hypotheses 1 will mistakenly show up as valid, although it is not. Rerunning the GP procedure with adapted settings increases this chance continuously. To account for repeated, multiple tests the so called [_Bonferroni adjustment_ or _Bonferroni correction_](http://www.aaos.org/news/aaosnow/apr12/research7.asp) has been proposed. The Bonferroni adjustment asks us to count the number of tests performed and at the end divide the statistical significance by this number. For example, if the confidence level is set at 95% and we conduct 5 tests, then the result is 95% / 5 = 19%. Be aware that "conducting a test" is actually not well defined in this context. If it means simply re-running the GP process, then this is still easy to compute. However, as many trading strategies are tested throughout each run it might be more natural to actually count every single trading strategy tested in all runs. Of course this would decrease the confidence level to such a low level that the whole data mining approach would be left useless. Authors like [Bailey et al. (2014)](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2308659) [1] are highly critical of backtested trading strategies, and their critique should not be taken lightly.

## Data snooping/lookahead bias

_Data snooping_ or the _lookahead bias_ refers to using data during backtesting at a point in time where this data would actually not have been available. Data snooping can either be an effect of programming errors or of invalid reasoning. A typical lookahead bias would be to calculate a technical indicator based on OHLC data at bar _t_ and then still allow a trade at the same bar. Either the trade must be made at the open of the next bar _t+1_, or the current bar's high, low and close are not yet available to calculate the indicator.

## Predictability tests

Some authors dispute the predictability of many financial time series altogether, often referring to the efficiency of market hypothesis. Biondo et al. (2013) [2] compare different random trading strategies with others based on technical indicators for indices such as the FTSE, DAX and the S&P 500. Not only do they come to the conclusion that there is little use in technical trading rules, but also - based on a Hurst exponent analysis - that none of these time series is likely to be predictable at all.  
Other authors take a less critical stance. Chen and Navet (2007) [3] &#40;who by the way have both published numerous papers on GP for trading strategy development&#41; for instance believe that some markets might indeed be efficient and thus inherently unpredictable, but others might not. Furthermore, the same market might actually be efficient/unpredictable at some times and inefficient/predictable at others. They suggest using statistical pre-tests to examine the situation.

* __Equivalent Intensity Random Search__: This is one test mentioned in Chen and Navet's paper. The idea is to compare the evolutionary procedure to a random search of equal intensity. The null-hypothesis to be rejected is that the evolutionary process is not better than the random search. If the evolutionary procedure is able to find trading strategies significantly better than the best randomly found search strategies then we have to conclude that indeed something can be learned from the time series by applying GP, and therefore reject the null-hypothesis.
* __Lottery Trading__: Another test mentioned in Chen and Navet's paper. Theoretically, the best GP evolved trading strategy should significantly beat a trading strategy using random buy and sell signals with a comparable trading intensity.
* __Runs Test__: A description of the _Runs Test_ can be found in a book by Vanstone and Hahn (2010) [4] or in this [online course of Pennstate University](https://onlinecourses.science.psu.edu/stat414/node/233). The basic idea is to measure if a "movement" in one direction is followed by another "movement" in the same or in the opposed direction. This principle can be applied equally on measuring the distributions of winning and losing trades or on price movements.
* __Serial Correlation Test__: This test is described in a paper by Escanciano and Lobato (2008) [5]. The basic idea is not very different from the Runs Test. A time series following a random walk has no auto-correlation (that is, it has no correlation between one time step and the next). If in a time series a significant auto-correlation can be found, then it must be concluded that this time series does not follow a random walk. Fernandez, whom I mentioned further above, also has a [short article on how to apply this test](http://mechanicalforex.com/2014/07/using-r-in-algorithmic-trading-testing-whether-an-instrument-follows-a-random-walk.html) on time series data using [R](http://www.r-project.org/).
* __Hurst Exponent Test__: This test relies on research conducted by Joseph Mandelbrot. Mandelbrot found that [financial time series have a _memory effect_](http://arxiv.org/abs/1110.5197). The Hurst exponent tries to measure the strength of this memory effect. There are different methods how to calculate the Hurst exponent. Some introductory material can be found in this short [article on Dukascopy](http://www.dukascopy.com/fxcomm/fx-article-contest/?The-Hurst-Exponent-Background-Methodologies&action=read&id=1950) and in [another one on analytics-magazine.org](http://www.analytics-magazine.org/july-august-2012/624-the-hurst-exponent-predictability-of-time-series).

----

## Other articles

### Previous
* Part 2: [Evolving Trading Strategies With Genetic Programming - Encoding Trading Strategies]({% post_url 2014-09-03-evolving-trading-strategies-with-genetic-programming-encoding-trading-strategies %})

----

# References

[1] Bailey D. H., Borwein J. M., Lopez de Prado M., Zhu Q. J. (2014): [Pseudo-Mathematics and Financial Charlatanism: The Effects of Backtest Overfitting on Out-of-Sample Performance](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2308659). Notices of the American Mathematical Society. No. 61(5). p. 458-471  
[2] Biondo A. E., Pluchino A., Rapisarda A., Helbing D. (2013): [Are Random Trading Strategies More Successful than Technical Ones?](http://arxiv.org/abs/1303.4351) Available at arXiv.org at [http://arxiv.org/abs/1303.4351](http://arxiv.org/abs/1303.4351).  
[3] Chen S.-H., Navet N. (2007): [Failure of Genetic-Programming Induced Trading Strategies: Distinguishing between Efficient Markets and Inefficient Algorithms](http://nicolas.navet.eu/publi/SHC_NN_Springer2007.pdf). In: Chen S.-H., Wang P. P., Kuo T.-W. (editors; 2007): Computational Intelligence in Economics and Finance. Springer-Verlag Berlin. p. 169 - 182  
[4] Vanstone B., Hahn T. (2010): [Designing Stock Market Trading Systems - With and Without Soft Computing](http://www.harriman-house.com/book/view/150/trading/bruce-vanstone-and-tobias-hahn/designing-stock-market-trading-systems/). Harriman House Ltd, Petersfield UK.  
[5] Escanciano J. C., Lobato I. N. (2008): [An Automatic Portmanteau Test for Serial Correlation](http://www.eea-esem.com/files/papers/EEA-ESEM/2008/210/joejan2008.pdf). Journal of Econometrics. Vol. 151, No. 2. p. 140 - 149
---
layout: post
title: Evolving Trading Strategies With Genetic Programming - Fitness Functions
comments: true
tags: [genetic programming, trading]
---
# Part 5
At the core of every genetic programming (GP) strategy is the _fitness function_. The fitness function specifies what the whole evolutionary process is looking for. Every individual is assigned a _fitness value_, which is computed by the fitness function. Individuals with a high fitness value stand a higher chance to be selected for reproduction and thus to create offspring. Finding a "good" fitness function is one of the most important design aspects of the development process. It is rarely the case that the first idea for a fitness function already produces great results, and defining one requires quite a deep understanding of the problem domain.<span class="more"></span> The following list contains a few necessary design decisions:

* Minimizing vs. maximizing fitness values
* Single-objective vs. multi-objective
* Normalization of fitness values
* Assigning weights to individual components of the fitness function

Let's take a closer look at each necessary decision.

## Maximization vs. minimization
Fitness functions can of course maximize certain target measures or they can minimize them. Typical target measures to maximize could be _total return_, _expected value_, _average size of winning trades_ or _hit rate_. Typical target measues to minimize could be _maximum drawdown_, _maximum number of consecutive losing trades_, or the _equity curve's volatility_. A common situation is to maximize some and minimize other fitness values at the same time.

## Single-objective vs. multi-objective
A fitness function with a single objective tries to maximize (or minimize) a single fitness value. This fitness value can possibly be the result of a mathematical formula combining multiple individual components, for example <div class="message">Maximize the total return divided by the maximum drawdown.</div> The higher the _total return_ in this formula the higher also the fitness value. The releationship between fitness value and and _maximum drawdown_ is inverse: the lower the _maximum drawdown_ the higher the fitness value. Of course the formula can lead to negative fitness values if _total return_ is actually a negative number. It is also presumed that _maximum drawdown_ can only take positive values. In other words, components of fitness value typically are valid only in a predefined range. As a consequence the fitness value itself is also valid only in a predefined range. For example, if _total return_ is valid in a range _(neg. infinity, pos. infinity)_ and _maximum drawdown_ in a range _(0, pos. infinity)_ the fitness values are automatically in the range of _(neg. infinity, pos. infinity)_. Of course the computer does not really understand the concept of infinity, and indeed situations might occur where it is necessary to prevent against overflows.

In the sample fitness function above, although possible it would be a bad idea to use negative values for the _maximum drawdown_ measure (arguing that a drawdown can be interpreted as a negative return) as this would be quite confusing.

In this article I will continue to refer to such "combined fitness functions" as single-objective, because the final fitness value is a single value. In contrast, multi-objective fitness functions do not try to aggregate multiple target measures into a single fitness value, but directly work on the multi-dimensional search space without reducing the number of dimensions. Multi-objective fitness functions for GP can be quite complicated to implement, yet from my experience they can actually lead to superior results compared with single-objective ones. I already mentioned two such fitness functions in an earlier article: the [non-dominated sorting genetic algorithm](http://www.cleveralgorithms.com/nature-inspired/evolution/nsga.html) (NSGA) and the [strength pareto evolutionary algorithm](http://www.cleveralgorithms.com/nature-inspired/evolution/spea.html) (SPEA) both of which I consider to be very powerful. These algorithms actually operate directly in a multi-dimensional "fitness landscape" without reducing the landscape into a one-dimensional "fitness number ray" as combined single-objective fitness functions do. The reader should also be aware that both algorithms by their design predetermine the GP selection and mutation operators. Working implementations for both algorithms can actually be found in the [ECJ library](http://cs.gmu.edu/~eclab/projects/ecj/).

## Normalization of fitness values
Sometimes it is necessary to normalize fitness values to a predetermined range. This might for example be the case if a few ouliers exist that are very far from the other fitness values. If a _fitness proportional selection operator_ is applied, these outliers could easily dominate all others, which is rarely desired. An alternative would be to use _rank-based fitness values_ as they do not suffer from such a problem.

Another problem is the relative difference in sizes of the involved fitness value components. Consider the following fitness function:
<div class="message">Maximize the total return divided by the average loss.</div>
In a situation where the value of the divisor is significantly greater or smaller than the dividend (as in the rule _maximize total return divided by average loss_), changes in one measure will always dominate changes in the other. If _total return_ is 50'000$ then a 2% change will result in +/- 1000$ for the divisor. Assuming the average loss in the observed trading system is 250$, then a 2% change is +/- 5$. Because +/- 1000$ always by far exceeds +/- 5$, in this example the nominator always dominates the denominator. Even a quite dramatic decrease of the average loss from 250$ to 125$ (average loss is reduced by 125$ or 50%) has much less impact compared to changes in the total return in this formula.

For these reasons (especially when using a combined single-objective fitness function) normalization of the individual components is usually necessary. There are three different alternatives:

__1. No normalization__  
If all components in the fitness function can be expected to have similar distributions (i.e. similar mean and standard deviation), then a normalization might not be necessary. This is rarely the case, however. Outliers are in fact a common phenomenon.

__2. Normalization of fitness value components with proportionate distances__  
The best fitness value component receives a predefined value of 1.0, the worst one 0.0. All other fitness value components are in proportionate distance between the two. (Koza favored a reversed order with the best fitness value component receiving the value 0.0 and the worst 1.0. The corresponding task is then to minimize this component rather than maximize it.) This procedure is still problematic if outliers exist, but sometimes - depending on one's choice of selection operators - it might be desired to preserve the relative distances between values.

__3. Normalization of fitness value components by first ordering all components according to their rank__  
Like in the previuos procedure the best fitness value component receives a value of 1.0 and the worst 0.0 (or vice versa). All others are in equal distance to each other according to their rank. Unlike the last procedure this one also resolves the problem of outliers. Yet, relevant information concerning relative distances between fitness value components is lost irrevocably.

## Common pitfalls
From my experience there are a few common pitfalls for beginners concerning the design of fitness functions. A typical beginner is likely to try apply a single-objective fitness function and try to maximize total return. This is easily understandable. After all, ultimately it's the amount of cash your trading strategy generated, is it not? As it turns out this approach rarely leads to good results. Although the logic applied is valid, there are all sorts of unsolved problems with maximizing total return. Here are a few points to consider:

__Strategy depends on very few trades__  
In a market of increasing prices it is often difficult to beat the market and generate real alpha. A strategy of buy and hold has the advantage of having very low trading costs. Therefore, in this situation it is not uncommon for the best performing trading strategy to simply buy in the beginning and hold till the end. The best evolved trading strategies correspondingly will probably have none to very few trades - which is just consequent, but still not very desirable. These trading strategies do not represent a repeatable way to success. They basically just perform by avoiding trading costs. Be aware that such "lazy trading strategies" also can imply a higher volatility than you are prepared to accept. This is because they simply repeat the market price developments due to their buy and hold tactics.  
For this reason, it might be interesting to try out an improved version of the fitness function which corrects for the number of trades, for example the _sum or product of the normalized total return and the normalized total number of trades_. Adding weights to each fitness component might also be of interest.

__Strategy depends on few winning trades__  
A related problem is to evolve strategies which rely heavily on a few very selected winning trades compared to many losing trades. Theoretically, this is a sound strategy, as long as the overall expected value is still positive (see further below). However, one needs to be very cautios. If the winning trades are too few, then the strategy might again not represent a repeatable way to success. In other words, the distribution of the winning vs. the losing trades is important too. If you went short the Dow Jones Industrial Average (DJIA) before the [Black Monday of 19th Oct 1987](http://en.wikipedia.org/wiki/Black_Monday_(1987)) you could have made a fortune. (The DJIA dropped by 22.61% on that day alone.) If you have only a single trade like this in your backtested strategy it might still make up years of small losses. But since a movement of this magnitude statistically occurrs only very, very rarely you cannot rely on it strategically. (Nevertheless you must be prepared for it to happen on the downside, otherwise such a loss could effectively wipe you out.) Be aware that with the increasing frequency in the occurrence of [flash crashes](http://en.wikipedia.org/wiki/Flash_crash) this problem has rather increased than decreased.

__Outliers dominate in fitness-proportional selection__  
Another problem is the existence of outliers. It happens relatively often that a few individuals are so much superior to all other individuals in the generation that they tend to dominate all the others although they do not represent a global optimum but only a local one. In case fitness-proportional selection operators instead of rank-based selection operators are used, outliers have much higher chances for being selected for reproduction. Re-running the evolutionary process with the same random seed (assuming a single-threaded application without runtime conditions) will of course just repeat the outcome, hence re-running with varied random seeds is recommended.  
This can happen both with single- as well as multi-objective fitness functions. Sometimes, in multi-objective fitness functions an individual's fitness might be located at the very "border" of the fitness landscape, with for instance one fitness component being the maximum of all values and the other one being zero. It is often helpful to plot the relative distribution of the fitness values to get an impression.

### Suggested fitness measures
This is a list of fitness measures I personally consider worth trying out in combination. I'd probably start with single-objective fitness function only maximizing expected value. Later on, once multi-objective fitness function is in place, I'd add other fitness components too.

__Maximize expected value (EV)__: The formula is <code>EV = Average Win * p<sub>Win</sub> - Average Loss * p<sub>Loss</sub></code>, with _Average Win (Loss)_ being the average return of a winning (losing) trade, and _p<sub>Win</sub> (p<sub>Loss</sub>)_ the probability for a winning (losing) trade. Of course the relationship p<sub>Win</sub> = 1.0 - p<sub>Loss</sub> must hold. The expected value _must_ be a positive number - if it is not, then the system will surely lose money in the long run! I consider this to be the most straight-forward and intuitive measure to maximize for every trading system. (For zero-sum trades, i.e. trades that neither generate nor lose money, I'd count them as losers nevertheless, as there is nearly always a "risk-free interest" alternative to which the money could have been assigned to.)

__Minimize max drawdown__: Nobody likes losing money in the markets. One common problem is to know when to shut down a trading strategy because it supposedly no longer works. Trading is thus always also a psychological game, do you really trust your strategy? Having big drawdowns might explode your account and lead to margin calls at the worst possible moment.

__Maximize number of trades__: This is a tricky one that beginners might not come up with easily. Every trade has its fees. Sometimes GP might assign very high fitness functions to trading strategies with only very few (but all winning) trades. Such strategies are artifacts based on random behavior as they do not represent meaningful, reproducible trading success. Maximizing the number of trades in combination with perhaps expected value is a counter-measure against this problem. Nevertheless this measure might be problematic, as it gives preference to many short-termed trades. When using this fitness measure it is therefore imperative to account for trading fees and possibly slippage.

__Maximize total return__: Although very simple and intuitive, maximizing total return is often only a good idea in combination with other fitness measures.

__Minimize GP tree size__: This is a tricky one too. The stated goal is of course to prevent code bloat leading to overfitting. We will in a later article cover parsimony pressure. I personally believe that trying to punish complexity of GP tree rules in the fitness function does not lead to very good results. Of course, this measure could also be combined with others, e.g. _maximize total return divided by GP tree size_. Still I have my concerns. I believe that parsimony pressure techniques which "mechanically" prevent the construction of large GP decision trees are far superior to punishing complexity in fitness functions.


----

## Other articles

### Previous

* Part 4: [Evolving Trading Strategies With Genetic Programming - GP Parameters and Operators]({% post_url 2014-11-01-evolving-trading-strategies-with-genetic-programming-gp-parameters-and-operators %})

### Next

* Part 6: [Evolving Trading Strategies With Genetic Programming - Punishing Complexity]({% post_url 2015-01-14-evolving-trading-strategies-with-genetic-programming-punishing-complexity %})

----

# References

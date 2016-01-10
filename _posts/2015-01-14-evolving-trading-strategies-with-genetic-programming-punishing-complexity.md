---
layout: post
title: Evolving Trading Strategies With Genetic Programming - Punishing Complexity
tags: [genetic programming, trading]
comments: true
---
# Part 6
One of the most poorly understood and yet at the same time most important concepts of genetic programming (GP) is _parsimony pressure_. It has long ago been demonstrated that for every type of statistical time series a function can be invented that arbitrarily well matches the observed values in the given time frame if that function is just complex enough. Yet, such a function is effectively worthless. As soon as new observations are added or as soon as predictions should be made for values lying outside the observed time frame the function terribly fails to deliver any meaningful result. I am of course talking about the problem of _overfitting_.<span class="more"></span>

Just take a second and think about it. Which one do you consider the more powerful statistical tool: (multiple) linear regression analysis or (multiple) "non-linear" regression analysis, e.g. based on polynomials? Many beginners probably believe that non-linear regressions should be more powerful than simple linear ones. Theoretically, this is true. But in practice there are at least two grave problems. First, it is rarely the case that you have a clear idea of the type of relation between the independent and the dependent variable(s). Second, allowing anything else than a linear combination of independent variables can easily lead to overfitting thus effectively reducing the predictive power of your regression formula.

When fitting a function to a set of observations, the goal thus must be to create a _robust_ function. In most cases such a robust function does not fit perfectly the given data, but it fits "well enough". Of course, making a judgement is difficult. But this is the case with every attempt to create a robust trading strategy, not just when using GP. Nevertheless, GP aggravates the problem of overfitting. The more elements a function consists of, that is the more complex it is, the more complicated time series can be fitted. In GP, trading strategies fitting the training data better receive a higher fitness value. The consequence is that more complex functions are naturally preferred by GP if no counter-measures are taken. These more complex functions have again a higher chance to create offspring of which again the most complex ones have the highest chance to fit the historical data. The result is _code bloat_, in other words highly complex functions overfitting historical data.

Thus, complexity must be punished. In GP, counter-measures taken to prevent against overfitting are called _parsimony pressure_. To the best of my knowledge there are basically only two different approaches in applying parsimony pressure:

1. Giving a lower fitness value to more complex functions.
2. Preventing overly complex offspring to be bred.

Before we look into each one it should be understood that no matter what parsimony pressure techniques we apply, ultimately it is the designer's decision what constitutes a robust trading strategy. There are no clear-cut guidelines, no simple measures to state exactly when a function is overfitting historical data. For this reason it is important to use out-of-sample data and to forward-test the strategy on live, never seen before data before starting to trade with real money.

### 1. Lowering the fitness value of complex functions
A rather obvious idea is to punish complexity by giving a lower fitness value to more complex functions. For instance, one might measure the number of nodes or the maximum depth in an evolved GP decision tree and divide something like the expected value by this measure. A higher number of nodes or a larger tree depth naturally leads to a lower fitness value. As demonstrated [in my last article]({% post_url 2014-12-22-evolving-trading-strategies-with-genetic-programming-fitness-functions %}), if the nominator and the denominator are very different from each other, then changes in one might dominate changes in the other. This must be paid respect to. One might question whether measuring tree depth or tree size actually is appropriate when it comes to the complexity of a trading strategy, but can be difficult to come up with a better idea. Unfortunately, both measures require a full traversal of the GP tree which significantly slows down the evolutionary process.  
The real disadvantage though is that from my experience this parsimony pressure technique does not perform too well. Often I experienced that despite my attempts nevertheless code bloat was observed. In contrast, using the next type of parsimony pressure techniques gave me much better results.

### 2. Preventing complex offspring to be bred
Another approach to parsimony pressure is to mechanically prevent the creation of offspring beyond a maximum number of allowed nodes. Typically, if a child is created where the number of nodes (or maybe the maximum node depth) exceeds an allowed maximum, then this individual is rejected and another one is created. ECJ, to give an example, lets you specify the number of attempts to be taken. Here, complexity is not punished by assigning it a lower fitness value (although this could be implemented additionally). Instead, the means used are "mechanical", it is simply not possible to create offspring exceeding beyond a certain node size. The exact details of this approach are beyond this article's intended scope and I really recommend taking a closer look to an implementation like ECJ. This approach worked pretty well for me, and I believe it to be superior in terms of result to the first one.

## Overfitting beyond code bloat
There is yet another problem. Even if we manage to control code bloat, another form of overfitting often arises, if we want to call it overfitting at all. Assuming that we already control code bloat, it happens still quite often that GP is able to evolve a trading strategy that looks quite promising on historical in-sample data but that simply falls short on out-of-sample data. That does not necessarily mean that the evolved strategy itself suffers of code bloat and is highly complex. On the contrary, it might even be a quite simple one. The overfitting we observe is of a different kind. Apparently, the function (trading strategy) evolved for the past data simply is not of use for out-of-sample data. However, as this is not the effect of code bloat, the only conclusion we are left with is that apparently what has worked in the in-sample period does not work in the out-of-sample period. This is a frustrating situation for which I have no solution. Running predictability tests might give a clue whether the markets are supposedly predictable or not, but still this is no guarantee. Markets do change, and it is a common phenomenon that trading strategies that worked in the past do no longer work today. Maybe at some point in the future they will start to work again. In such a situation it is the best to just accept the state of affairs and continue to another promising market.

----

## Other articles

### Previous

* Part 5: [Evolving Trading Strategies With Genetic Programming - GP Parameters and Operators]({% post_url 2014-12-22-evolving-trading-strategies-with-genetic-programming-fitness-functions %})

### Next

* Part 7

----

# References

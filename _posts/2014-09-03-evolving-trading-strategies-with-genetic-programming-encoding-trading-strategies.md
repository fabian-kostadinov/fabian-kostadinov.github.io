---
layout: post
title: Evolving Trading Strategies With Genetic Programming - Encoding Trading Strategies
comments: true
---
# Part 2
As I have shown in a [previous post]({% post_url 2014-09-01-evolving-trading-strategies-with-genetic-programming-an-overview %}) in GP entry and exit decision rules are encoded in a tree form. The decision rule tree returns a boolean value for every processed bar, which is interpreted as an entry (root node returning <code>true</code>) or exit (root node returning <code>false</code>) signal. Some authors suggest using a single rule tree for entry and exit signals, but I personally prefer evolving dedicated rule trees for both entry and exit rules, as I believe them to produce better signals.<!--more--> During strategy evaluation the program alternately "reactivates" one or the other tree and ignores the signals produced by the inactive tree. This works for long-only or short-only trading. For long/short-strategies this approach leads to four different decision rule trees being evolved. This obviously also increases the degrees of freedom of our trading strategies and therefore the danger of overfitting.

## Typed nodes
I will not explain here in detail how the GP mutation and crossover operators work on decision rule trees as descriptions can be found in various books on GP (see for example Poli et al. 2008 [1]). In very basic GP implementations nodes in the decision trees are untyped. This is problematic as in such a situation mutation and crossover tend to create invalid decision trees with a high probability. Let us imagine that a moving average rule requires a lookback period size in the form of an integer value and a price series to operate on. A naive implementation of the mutation operator might simply randomly exchange the integer node with another technical indicator. Example: <code>EMA(Close, 12)</code> becomes <code>EMA(Close, ROC)</code> through random mutation. The mutated moving average node now receives a price series and another technical indicator as its inputs which results in invalid, meaningless code. The same problem also applies for the crossover operator.

Some people simply try to repeat running these operators until a valid solution is found by luck. However, as should be obvious, this is an extremely inefficient approach. A much better alternative is to introduce types for each node. By introducing input and output types for all nodes we make sure that the mutation and crossover operators can only chose from a predefined set of allowed alternatives, thus reducing inefficiently spent search time to an absolute minimum. This of course comes at a price. Implementation of typed nodes in GP is much more complicated than simply using untyped nodes. This is one more reason to rely on readily available GP libraries including this functionality instead of trying to implement everything from scratch anew.

As a consequence, the architecture of decision trees naturally follows a certain logical order.

1. In the upper section of the tree nodes typically have the same return types as the root node, i.e. boolean values or a *buy/sell/do_nothing* signal which is aggregated in the root node.
2. In the upper middle section of the tree we often find nodes that are able to "convert" from one input type to another output type, for example the <code>>, <, >=, <=</code> or <code>crosses above/below</code> nodes which all take numeric or price inputs and return boolean outputs.
3. In the lower middle section we find the technical indicator rules, for example moving averages, oscillators and trend indicators based on open/high/low/close/volume (OHLCV) bar data.
4. The bottom section of the tree, the leave nodes, entirely consist of either ephemeral random constant (ERC) nodes or price series data nodes.

Defining the return types of nodes requires careful reflection. It is my impression that many developers fail to understand this crucial point. Consider the following decision rule:

<code>(high<sub>t-1</sub> - close<sub>t-1</sub>) > open<sub>t</sub></code>

Whereas at a first glance there is nothing wrong with this rule from a mathematical standpoint, a closer inspection exposes it as being quite meaningless. Why is this so? Because the comparison operator is applied on two completely different objects! On the left-hand-side we have a _difference of_ or a _distance between_ two prices, whereas on the right-hand-side we have a _price_. Both have a very different semantical meaning. For the sake of the example, let us assume that *high<sub>t-1</sub> = 12.00$*, *close<sub>t-1</sub> = 11.66$* and *open<sub>t</sub> = 11.78$*. We therefore effectively compare <code>0.34$ > 11.78$</code>. Obviously this rule will always return false with almost certainty, because the last bar's difference between the high and the close prices practically never exceeds the current open price! (One of the few real-world exceptions might be when a stock plunges down due to a bankruptcy announcement.) One must always be concerned with the question whether two nodes can meaningfully be compared to each other. Sometimes finding an answer is not easy.
  
Giving input and output types to nodes not only prevents the evolution of meaningless decision rules due to comparing incompatible return types, but also significantly decreases the already huge search space.

## Ephemeral random constants

Handling ERCs properly is another issue. Many technical indicators rely on OHLC(V) data and one or several integer or float constants. For example, a moving average has a lookback window size, which is encoded as an integer ERC. From the context we already know that this lookback window must be a positive value, all values <= 0 are disallowed. We can also in most cases define a meaningful maximum size for the same value, e.g. 500 trading days for a trading strategy with a daily trading frequency - having an even longer lookback period just does not make sense. In setting meaningful minimum and maximum sizes for ERCs we further restrict the search space. For floating point numbers, we could even only allow numbers from a pre-computed array, e.g. <code>[0.20, 0.21, 0.22, ..., 10.00]</code>. Such a design silently implies that all random mutations to a float ERC beyond a certain precision are effectively meaningless.

## Beyond entry and exit signals

More advanced trading strategies might not only consist of decision rule trees for long/short entry/exit signals, but additionally of (usually simpler) decision trees for position sizing or stop loss and take profit rules. This is an example for for a long position and a stop loss rule that is triggered if the price falls by more than 3% of the entry price:

<code>current price <= ((1.0 - 0.03) * entry price)</code>

The 3% value could of course be evolved using a float ERC. Common stop loss techniques include fixed price stops set at a certain distance from the entry price, percentage stops or average true range (ATR) stops. The same or similar techniques can be used for take profit rules.

## Technical indicators

Which technical indicator rules to select is largely a matter of taste. My assumption is that the most common indicators are also the best ones. That is, moving averages in general and MACD in particular, stochastic oscillators, A/D and Chaikin Oscillator, Directional Movement Index and so on are all promising canditates. There are many good sources of information about technical indicators and how to implement them, see for instance [stockcharts.com](http://stockcharts.com/school/doku.php?id=chart_school:technical_indicators) or Colby's _The Encyclopedia of Technical Market Indicators_ [2].  
When it comes to how to encode technical indicators the procedure is:

1. First, create a dedicated node per technical indicator you want to use. For example, although a MACD de facto consists of two moving averages nevertheless create a dedicated MACD node. The reason is simple. The chance for GP to evolve a powerful and complicated to compute technical indicator by itself is very small. If you however provide a dedicated MACD node right from the start as a possible candidate node the evolutionary process only has to select it from a list of possible alternatives. Of course nothing prevents you from additionally creating dedicated (simple, exponential, weighted etc.) moving average node types.
2. Next, decide upon the node's output type. With a MACD type you are probably interested in the faster moving average crossing above or below the slower moving average. The MACD node could therefore return a boolean value, indicating whether the fast moving average is currently above the slow one or not. Or you could consider returning an enumeration constant, indicating whether a the fast moving average has just crossed above, below or not at all (e.g. <code>UP</code>, <code>DOWN</code>, <code>NONE</code>).
3. Decide upon the node's inputs. A dedicated MACD node will certainly need i) an open or close price series to operate on, ii) a lookback period for the fast moving average, iii) a lookback period for the slow moving average. Maybe you also want to add an additional input node which decides upon which moving average type (e.g. simple vs. exponential) should be used internally of the MACD node.

Some traders apparently believe that GP or other data mining strategies have the power to miraculously "find" new technical indicators never heard of, which then magically create the most extraordinary returns. Sorry to disappoint you guys, but that just won't happen.

[TA-Lib](http://ta-lib.org/) is a good open-source library containing implementations for many technical indicators. It is available for a variety of programming languages such as C/C++, Java, Perl, Python and .NET. The only real drawback at the moment of writing is its lack of documentation.

----

## Other articles

### Previous

* Part 1: [Evolving Trading Strategies With Genetic Programming - An Overview]({% post_url 2014-09-01-evolving-trading-strategies-with-genetic-programming-an-overview %})

### Next

* Part 3: Evolving Trading Strategies With Genetic Programming - Data

----

# References
[1] Poli R., Langdon W. B., McPhee N. F., Koza J. R. (2008): [A Field Guide to Genetic Programming](http://cswww.essex.ac.uk/staff/poli/gp-field-guide/). Available online at [http://cswww.essex.ac.uk/staff/poli/gp-field-guide](http://cswww.essex.ac.uk/staff/poli/gp-field-guide/)

[2] Colby R. W. (2002): The Encyclopedia of Technial Market Indicators. Second Edition. McGraw-Hill, New York.
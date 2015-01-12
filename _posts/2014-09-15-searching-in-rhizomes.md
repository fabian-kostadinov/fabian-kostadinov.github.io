---
layout: post
title: Searching in Rhizomes
comments: true
tags: [rhizome]
---
In this post I will look into how it is possible to search in rhizomes. I will assume that the rhizome implementation relies on pairing functions to store relations. I described the basic necessary data structures [in a previous post]({% post_url 2014-09-09-implementation-of-rhizomes %}). The term "searching in a rhizome" is not defined precisely. Searching implies that some contextual order is given and that the search is conducted in relation to this order. For rhizomes (similar to graphs) it is not entirely clear what constitutes this contextual order, the the definition of contextual order might depend upon one's situation.<span class="more"></span>

Let us begin with a simple example and gradually continue to more complicated ones. First, let us assume we want to know whether a relation between the atomic symbols _A_ and _B_ exists. We can create the two relations <code>(A, B)</code> and <code>(B, A)</code>, look up the terminal relations for A and B and reduce them to their final z-pairing values.

1. <code>r<sub>(A, B)</sub> <= ((0, 0), (1, 1)) = (0, 3) = 9</code>
2. <code>r<sub>(B, A)</sub> <= ((1, 1), (0, 0)) = (3, 0) = 12</code>

Next, we simply look up each z-pairing values <code>9</code> and <code>12</code> in the hashmap of pairings. If there exists such a value, then the corresponding relation has been stored. One should note that the search can easily be parallelized.

Another example. This time we will look for a string of three atomic symbols, _A_, _B_ and _C_. For the sake of the example, let us assume we know the ordering of the symbols from left to right (_A_ followed by _B_ followed by _C_), but not their nesting. In this situation, two different possible pairings exist: <code>((A, B), C)</code> and <code>(A, (B, C))</code>. We compute all possible nestings, replace the atomic symbols with terminal relations, reduce them to their z-pairing values and look them up in the hashmap. We can then determine which nestings actually exist. If none does then the ordered string _ABC_ has not been stored. If we were only interested in the question if at least one nesting exists, but would not care which nesting this were, then we could immediately stop computation after we found the first one. Again, the whole process can be parallelized easily.

The situation becomes more complicated if we cannot guarantee the order between _A_, _B_ and _C_. Basically, we need to compute all possible orders and nestings and then look them up individually. These are all possible orderings and nestings:

* <code>((A, B), C)</code>, <code>(A, (B, C))</code>,
* <code>((A, C), B)</code>, <code>(A, (C, B))</code>,
* <code>((B, A), C)</code>, <code>(B, (A, C))</code>,
* <code>((B, C), A)</code>, <code>(B, (C, A))</code>,
* <code>((C, A), B)</code>, <code>(C, (A, B))</code>,
* <code>((C, B), A)</code>, <code>(C, (B, A))</code>

Fortunately, it is possible to memoize in-between results. Some relations are shared, for example <code>(A, B)</code> is part of both <code>((A, B), C)</code> and <code>(C, (A, B))</code>. If <code>(A, B)</code> does not exist, we can immediately stop computation for both cases and conclude that neither of these two more complex relations exist. We only need to compute all z-pairing values if none of the listed relations exists.

What if we do not only want to know _if_ a relation actually exists but also _where_ it is stored? As I mentioned further above as long as no clearly defined contextual order is given this is impossible to answer. A rhizome has no inherent order or indexing scheme. The term "where" is not defined. What is possible to answer though is questions of the type: _Does the relation represented by z-pairing value <code>z</code> contain the relation <code>(A, B)</code>?_ We simply need to expand the relator <code>r<sub>z</sub></code> to its relata and continue down the rhizome tree recursively. If anywhere in the chain the relation <code>(A, B)</code> is found the answer is yes.

----
## See also:

* [Implementation of Rhizomes]({% post_url 2014-09-09-implementation-of-rhizomes %})
* [A Short Introduction to Rhizomes]({% post_url 2014-09-08-a-short-introducton-to-rhizomes %})
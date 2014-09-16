---
layout: post
title: Pile or Rhizome? A Short Pamphlet.
comments: true
tags: [rhizome]
---
[Erez Elul](http://namzezam.wikidot.com/), whose honor it is to have "discovered" or "invented" the data structures I deliberately continue to call _rhizomes_, originally named his invention _pile_ (or _pile system_). The relatively few authors (for instance [Peter Krieg](http://en.wikipedia.org/wiki/Peter_Krieg), [Ralf Westphal](http://blog.ralfw.de/), [Ralf Barkow](http://ralfbarkow.wordpress.com/), Miriam Bedoni as well as others besides them), who both commented and contributed on the invention accepted this naming. I must say I never found the term particularly saying for several reasons.<!--more-->

* In my opinion the most significant characteristic of piles/rhizomes is universal connectivity. Everything can be connected to everything else. Quite differently, Erez Elul's reasoning apparently focused much more on the fact that relations are "piled up onto each other" - hence the choice of term. In my eyes, this is however not the distinguishing criterion. For example, also in binary trees nodes are "piled" onto each other in a sense. Yet, unlike piles/rhizomes, binary trees follow the traditional node/edge dichotomy. They lack the feature of universal connectivity.
* The term _pile_ is quite close to _heap_, and both could be confused easily. Runtime environments for programming languages such as the _Java Virtual Machine_ (JVM) internally use a datastructure called _heap_ to hold objects in memory.
* When I first by chance stumbled upon the term _rhizome_ in Gilles Deleuze's and F&eacute;lix Guattari's thinking, which they had borrowed from [botanics](http://en.wikipedia.org/wiki/Rhizome), I immediately found the term very fitting.
<blockquote>
Deleuze and Guattari use the term rhizome throughout their work, especially in their discussion of thought in "[A Thousand Plateaus](http://www.amazon.com/gp/product/0816614024)." They argue that traditional thought is tree-like, in that it follows a linear pattern, branching off at various points. Rhizomes, taken from a kind of [root system](http://en.wikipedia.org/wiki/Rhizome) found in nature, are non-linear, and non-hierarchical.
</blockquote>
([Source](http://www.critical-theory.com/rhizome/))  
Rhizomes are thus a philosophically well described concept, and a corresponding theory in computer science could possibly profit from earlier work done by these post-structuralists.

Of course there is no ultimate truth in either name choice, and there are good reasons to stick to the customary term _pile_.
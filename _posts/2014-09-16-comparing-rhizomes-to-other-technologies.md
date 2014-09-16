---
layout: post
title: Comparing Rhizomes to Other Technologies
comments: true
tags: [rhizome]
---
At a first glance, rhizomes may have a lot in common with existing technologies. Yet, when taking a closer look, there are important differences, and it is not possible to simply reduce a rhizome to one or another existing technology. In this post I will quickly compare rhizomes to a variety of different mathematical and computational concepts and data structures.<!--more-->

## Binary Decision Diagrams
Here is [Wikipedia's definition of binary decision diagrams (BDD)](http://en.wikipedia.org/wiki/Binary_decision_diagram):
<blockquote>
In computer science, a binary decision diagram (BDD) or branching program [...] is a data structure that is used to represent a Boolean function. On a more abstract level, BDDs can be considered as a compressed representation of sets of relations. Unlike other compressed representations, operations are performed directly on the compressed representation, i.e. without decompression.
</blockquote>
A BDD is a deviant of a binary tree. That is, in a BDD every node (except leaves) is connected to two children. Every node encodes a (sub-) function which returns either <code>1</code> (i.e. true) or <code>0</code> (i.e. false). Depending on the function output, the tree's traversal is continued at the corresponding child until a leave is found. In this way, a path chosen through the tree and the whole tree eventually returns a bit string. BDDs have been used extensively in CAD software and also for formal verification purposes. More recently, alternative usages have also been proposed. (There's an [online lecture of Donald Knuth - the man himself! - on BDDs at Stanford University](http://myvideos.stanford.edu/player/slplayer.aspx?coll=ea60314a-53b3-4be2-8552-dcf190ca0c0b&co=18bcd3a8-965a-4a63-a516-a1ad74af1119&o=true).)

Rhizomes are structurally similar to BDDs. The most important difference though is that BDDs are basically graphs whereas rhizomes are not. For example in BDDs it is not possible (and does not make much sense...) to connect a node to an edge or two edges with each other due to the underlying node/edge dichotomy. Rhizomes do not know such restrictions, because the only existing building block are relations. This means that every BDD can be implemented as a rhizome, but not every rhizome can be implemented as a BDD.

## Set theory
Set theory has been very fundamental to the design of (so called) relational databases. Set theory prescribes a hierarchy of sets and elements. The _inclusion_ operator &#8712; defines that an element _e_ belongs to or is included in or is an element of a set _S_. The reverse is usually not allowed: _S_ &#8712; _e_ is false or invalid in most real-world applications. For example, in a relational database a row cannot relate to its own table. (We could add the table name as an attribute to a row, but this would not be safe, because the table could be renamed without the row taking notice.)

Again, in rhizomes there are no such restrictions. As was [stated in a patent (later on withdrawn) by Erez Elul/Pile Systems](http://www.google.com/patents/US20060155755), inclusion is only a special form of connection. Every inclusion is a form of connection, but not every connection is an inclusion. Therefore rhizomes operate on more powerful concepts than set theory.

## Object-oriented programming
Similar arguments as for set theory also apply to object-oriented programming (OOP). To give an example: Inheritance is only one form of connection, but connection is a broader concept than inheritance. The same can be said about specification, usage and other types of relations between objects. Rhizomes allow all of these relations, OOP allows only these relations.

## Lisp/Clojure
Lisp and Clojure are sometimes said to be _homoiconic_. [Homoiconicity](http://en.wikipedia.org/wiki/Homoiconicity) refers to the fact that in these programming languages all data structures can be either interpreted as lists or as algorithms because the language's syntax for both is the same. For example, the statement <code>(+, a, 3)</code> can indicate a list of three elements as well as the mathematical expression "apply the + operator on the variable a and the integer 3". Rhizomes are also homoiconic, because every relation _r<sub>k</sub> <= (r<sub>i</sub>, r<sub>j</sub>)_ both expresses a relational fact (data) as well as an algorithm that either reduces the relation to its z-pairing value _r<sub>k</sub>_ or expands the relata _r<sub>i</sub>_ and _r<sub>j</sub>_ further to ordered pairs. Of course, Lisp and Clojure are both complete programming languages, whereas rhizomes are - at least currently - not.

I am not an expert on this, but at least Clojure treats lists internally as immutable data structure. If we remove an element and add a new one, effectively a new list is created as a deviant from the original one. This implies that elements in a list are never really replaced or deleted. If a list with other elements is produced at runtime, this is in fact "a different" list. This is comparable to rhizomes, where deletions are unnecessary. If a relatum _r<sub>m</sub>_ is paired with another relatum _r<sub>n</sub>_, and then it is re-paired with _r<sub>o</sub>_, then _(r<sub>m</sub>, r<sub>n</sub>)_ and _(r<sub>m</sub>, r<sub>o</sub>)_ are in fact different relations. Their corresponding z-pairing values are not the same.

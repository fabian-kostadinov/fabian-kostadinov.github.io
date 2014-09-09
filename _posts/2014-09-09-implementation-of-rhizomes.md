---
layout: post
title: Implementation of Rhizomes
comments: true
---
There are at least three different ways how to implement a rhizome on existing hard- and software platforms: as an object tree, relying on a programming language pointer arithmetic, or using mathematical pairing functions.

## Implementation as object tree

The most obvious option is to re-interpret a rhizome as an object tree and use an object-oriented approach to implementation. This is probably the easiest way to quickly come to a working solution. This works for a relatively small number of relations, but not for big numbers.

## Implementation based on pointer arithmetic

A more sophisticated option is to use a programming language with pointer arithmetic capabilities such as C/C++ or C#. This was indeed Erez Elul' and Miriam Bedoni' choice when implementing the [first version of their _Pile Engine_](http://sourceforge.net/projects/pileworks/). (I personally prefer to use the term _rhizome_ for _pile_ as I deem it more expressive.) There is also [another version by Ralf Westphal (?)](https://code.google.com/p/pile/), which I think is different from the first. Unfortunately, it is hard to find more information other than a few only partially functional links:

* [Ralf Westphal's blog entry](http://weblogs.asp.net/ralfw/441384)
* [Ralf Barkow's blog entry](http://ralfbarkow.wordpress.com/2006/04/23/what-is-a-pile_object/)

Although this is a highly viable option, it can become quite tricky to solve the implementation details, for example to parallelize the whole implementation.

## Implementation based on pairing functions

There is however a third, highly interesting alternative, described in a short [blog post by Ralf Barkow](http://ralfbarkow.wordpress.com/2006/06/21/the-cauchycantor-diagonal-method/). The idea is to use a mathematical _pairing function_ for our implementation. A [pairing function](http://en.wikipedia.org/wiki/Pairing_function) is a mathematical function taking two numbers as an argument and returning a third number, which uniquely identifies the pair of input arguments. It is always possible to re-compute the pair of arguments from the output value. Two pairing functions are currently known to me.

__Cantor pairing function__:

{% highlight js %}
pair(x, y) = z = 1/2 * (x^2 + 3x + 2xy + y + y^2)

unpair(z) = (x, y) = {  
    x = z - (q * (1 + q))/2,  
    y = (q * (3 + q))/2 - z },  
    with q = floor((-1 + sqrt(1 + 8z))/2)
{% endhighlight %}

__Szudzik pairing function__:

{% highlight js %}
pair(x, y) = z = {  
    x^2 + x + y, if x = max(x, y)  
    y^2 + x, otherwise  
}

unpair(z) = (x, y) = {
    x = z - floor(sqrt(z))^2, y = floor(sqrt(z)), if z - floor(sqrt(z))^2 < floor(sqrt(z))
    x = floor(sqrt(z)), y = z - floor(sqrt(z))^2 - floor(sqrt(z)), otherwise  
}
{% endhighlight %}

Using one of the above pairing functions, we can now assign non-negative integer values to relators and relata. The relator _r<sub>k</sub>_ is assigned the pairing value <code>z</code>, and the relata _(r<sub>i</sub>, r<sub>j</sub>)_ the ordered pair of values <code>(x, y)</code>. According to our definition, terminal relations are defined by "pointing to themselves", in other words both relata being the same: _r<sub>k</sub> <= (r<sub>i</sub>, r<sub>j</sub>)_ with _r<sub>i</sub> = r<sub>j</sub>_. Thus, it is strictly guaranteed that terminal relations are the only ones where <code>x = y</code>.

This figure shows an example for the Szudzik pairing function. 

![Szudzik pairing function example](/public/img/szudzik-pairing-func.jpg "Szudzik pairing function example")

There are seven terminal relations <code>(0, 0)</code> to <code>(6, 6)</code> representing the characters _A_ to _G_. The grid shows the <code>z</code> pairing values for each ordered pair <code>(x, y)</code> according to the Szudzik pairing function. Let us assume we wanted to encode the String _ABC_ as two nested ordered pairs <code>(A, (B, C))</code>.

1. Replace all characters with their terminal ordered pair of values: <code>((0, 0), ((1, 1), (2, 2)))</code>.
2. Repeat replacing all ordered pairs with their z pairing value until there is only a single z value left: 1) <code>(0, (3, 8))</code>, 2) <code>(0, 67)</code>, 3) <code>4489</code>.

The relator _r<sub>4489</sub>_ encodes the nested ordered pairs <code>(A, (B, C))</code>. Nesting and order do matter, the result would be different for <code>((A, B), C)</code> or <code>(A, (C, B))</code>.

Of course the process works in a reverse way too.

1. Repeatedly expand all <code>z</code> pairing values by replacing them with an ordered pair of <code>(x, y)</code> values until you meet a pair where <code>x = y</code>: 1) <code>4489</code>, 2) <code>(0, 67)</code>, 3) <code>((0, 0), (3, 8))</code>, 4) <code>((0, 0), ((1, 1), (2, 2)))</code>.
2. Replace all ordered pairs by looking up their data items in the dictionary: <code>(A, (B, C))</code>.

We can now traverse the rhizome upwards and downwards (from relata to relator and vice versa) without the need to store any data. The grid shown above only describes an algorithm how to pair values, it is not stored anywhere as real data content. The only place so far where actual data is stored is for the mapping of atomic symbols to terminal relations. This is very attractive because the symbol/terminal-dictionary grows only linearly with an increasing number of terminals.

So how do we keep track of which non-terminal relations have already been established and which have not? We know for example how to compute the z-pairing value for the pair of symbols _(B, C)_, but how can we know if such a pairing has been established or not? For this purpose, we need another data structure. Its only task is to keep track of what pairings have been established and which have not. The simplest approach is to use a two-dimensional bit array equivalent to the logical grid above. If a bit is set in the 2d-array, this indicates that a connection exists between its indices <code>x</code> and <code>y</code>. This will result in a matrix which is quite densely populated for low <code>x</code> and <code>y</code> values but sparsely populated for high <code>x</code> and <code>y</code> values. Unfortunately, the matrix grows in size at O(n<sup>2</sup>). Even if we only store bits this can quickly eat up all our resurces. At least for the sparsely populated matrix area we should try to find leaner solution.

As an alternative, I suggest using one more hashmap (or possibly list) structure. The hashmap contains each relatum appearing in a relation as a key, and two lists as its value. One list contains all associative relata for this (normative) relatum, the other contains all normative relata for this (associative) relatum. The list themselves are stored as packed bit strings. The best bit string packing algorithms do not even require the bit string to be unpacked to check whether a certain bit is set or not, only for setting or deleting a bit unpacking and repacking is executed. (Daniel Lemire's blog contains some articles on [bit packing in C++](http://lemire.me/blog/archives/2012/03/06/how-fast-is-bit-packing/) and [Java](http://lemire.me/blog/archives/2013/07/08/fast-integer-compression-in-java/).) The list should be able to quickly return a list of relata this relatum is in a relation with. An algorithm I used personally is the _CONCISE algorithm_ ([Colantonio](http://ricerca.mat.uniroma3.it/users/colanton/publications.html) and Di Pietro [1]), which seems to even beat _Word Aligned Hybrid (WAH)_ bitmap compression [4] on which for example the [FastBit library](https://sdm.lbl.gov/fastbit/) [5], [6] is based. Colantonio wrote a highly efficient open source Java implementation of the CONCISE algorithm called _Extendedset_, which is available in two GitHub repositories [2], [3].

## Some remarks

Some general remarks. First, using pairing functions only works if all terminals are assigned ordered pairs where _r<sub>i</sub> = r<sub>j</sub>_. In the grid, all terminals are stored along the grid's diagonale. The reason is that a stop criterion is required when traversing the rhizome in a reverse way from relator to relata. Imagine that we would allow terminals to be stored also outside the diagonale. Then, for example, the character _C_ could be stored as <code>(0, 2)</code>. As both <code>0</code> and <code>2</code> appear as z-values (relators) in the grid, we cannot decide if the ordered pair <code>(0, 2)</code> already represents a terminal or must be unpaired even further to <code>((0, 0), (1, 0))</code>.

Second, the z pairing values can become quite large. With a linear growth of the x and the y values, the z values grow quadratically. Yet, at the same time, the expressive power or the information content of z-values also grows quadratically. At the same time, the need for larger z-values continuously decreases, as certain complicated combinations of terminals rarely occurr.

Third, this approach requires to decide upon a predefined set of terminals. It is impossible to decide a posteriori to split a terminal into several sub-relations. Whereas this sounds like a severe restriction, I believe that it is not on many cases. In most real-world examples there exists a naturally restricted vocabulary or set of atomic symbols, which we silently accept.

* For example in digital computing, the atomic symbols are the binary digits <code>0</code> and <code>1</code>. Or, taking a byte as an atomic symbol, the set of possible bytes contains only 256 different atoms.
* In language, character sets such as ASCII or UTF-8 provide a large number of atomic symbols letter and number symbols which are expressive enough for the vast majority of use cases.
* Natural language uses a limited - albeit large - number of words. Applying rhizomes on natural language processing we could actually create a combined set the most common words enhanced with the atomic letter and number symbols to create less common words.
* The periodic table consists of a few dozens of chemical elements.
* The human genomes consists of only four fundamental molecules, the guanine (G), adenine (A), thymine (T) and cytosine (C).

Fourth, because rhizome trees do not store data items but only relations, multiple dictionaries can be applied on the same rhizome tree at the same time. In other words, the interpretation of any z-value or ordered pair (x, y) is entirely the responsibility of the user and the context. The situation is the same to processing binary data. The interpretation of a bit string like <code>00001010</code> is entirely left to the computational context, which not only defines whether this bit string is to be read from left to right or from right to left but also whether it designates a character, number, color, screen position, i/o address or something else. The minimum requirement for each context is a dictionary mapping atomic symbols to terminal relations and a second map or list keeping track of the actual pairings.

----

# References

[1] Colantonio A., Di Pietro R. (2010): [CONCISE: Compressed 'n' Composable Integer Set](http://arxiv.org/abs/1004.0403). Paper submitted to arXiv.org. No. 1004.0403.

[2] [Extendedset: An open source Java implementation of the CONCISE algorithm authored by Alessandro Colantonio](https://github.com/tuplejump/extendedset/). Version 1.
 
[3] [Extendedset: An open source Java implementation of the CONCISE algorithm authored by Alessandro Colantonio](https://github.com/metamx/extendedset). Version 2.

[4] Wu K., Otoo E. J., Shoshani A. (2001): [A Performance Comparison of bitmap indexes](https://sdm.lbl.gov/~kewu/ps/LBNL-48975.pdf). Proceedings of the tenth international conference on information and knowledge management (CIKM '01). p. 559-561

[5] [FastBit library on Codeforge](https://codeforge.lbl.gov/projects/fastbit/).

[6] [FastBit library on GitHub](https://github.com/gingi/fastbit).
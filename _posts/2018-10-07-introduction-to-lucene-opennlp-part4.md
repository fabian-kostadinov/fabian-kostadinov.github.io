---
layout: post
title: Introduction to Lucene 7 OpenNLP - Part 4
comments: true
tags: [lucene, opennlp]
---
I have written three blog posts about how to use Lucene 7 and OpenNLP to index part-of-speech tags and then use phrase queries to search on these tags. What I haven't shown so far is what's so cool about having such a capability.<span class="more"></span>
Imagine that you are building a search engine containing various articles. You are interested to know what nouns a particular indexed term is typically paired up with. For example, the noun "heart" could be paired up with another consecutive noun such as "heart failure", "heart disease", "heart". Or it could be paired up with an adjective such as "broken heart", "lonely heart" etc. That's exactly what we can do now. We can mix up OpenNLP POS tags together with words in the same query. Let's assume we are looking for triplets in this form:
```
the expensive car
a slow car
a fancy automobile
a car park
the car accident
```
We can express these as four different types of queries mixing POS tags with the word car and automobile:
```
DT JJ car
DT JJ automobile
DT car NN
DT automobile NN
```
This would result in the following query assuming the relevant text is indexed in a field called "body":
```
+(body:"DT JJ car" body:"DT JJ automobile" body:"DT car NN" body:"DT automobile NN")
```
We have also already seen how both a combination of phrase queries and boolean queries can be used to create such an expression, or as yet another option multi phrase queries.

As pointed out already earlier OpenNLP's POS tags are following the [Penn Treebank II tag set standard](https://www.clips.uantwerpen.be/pages/mbsp-tags). Depending on the use case these tags might be too fine-grained. For example there are four different POS tags for nouns (NN, NNP, NNS, NNPS), several ones for verbs, for pronouns and so on. Although there exists a [WildcardQuery](https://lucene.apache.org/core/7_4_0/core/index.html?overview-summary.html) class that allows us to build expressions such as NN*, wildcards can be computationally expensive. Wildcards are therefore not the best idea in this context.

A better option is to use our own simplified set of POS tags in the indexing step. For every POS tag output by OpenNLP a brief check is performed and the tag is replaced by a simpler form:
```
NN -> NN
NNP -> NN
NNS -> NN
NNPS -> NN
VB -> VB
VBZ -> VB
...
```
Remember the class [TokenFilter](https://lucene.apache.org/core/7_4_0/core/index.html?overview-summary.html)? We used one in our OpenNLPAnalyzer. For each input token [The][quick][brown][fox]... read from the token stream our token filter added the POS tag to the type field in the index: [DT][JJ][JJ][NN]... . We can simply wrap this filtered token stream again with yet another token filter. There are again different ways how to implement this, I'll leave this as an exercise to the reader. A good start to learn more about how to write a token filter is both the implementation of class [OpenNLPPosFilter](https://lucene.apache.org/core/7_4_0/analyzers-opennlp/org/apache/lucene/analysis/opennlp/OpenNLPPOSFilter.html) and [this short tutorial](http://nathanchen.github.io/14457412441370.html).

Maybe you are still wondering what exactly this all gives us. "It's a nice use case", you might be thinking, "but how is this relevant to anyone?" What it gives us is the ability to create taxonomies expressed as queries and then extract entities from documents. This is a pretty common use case for companies which have to process large numbers of text documents in an automated way. In fact, certain companies offering services in the space of natural language processing and AI are making money based on these or similar ideas.

## Other posts

### Previous
* Part 1: [Introduction to Lucene 7 OpenNLP - Part 1]({% post_url 2018-09-08-introduction-to-lucene-opennlp-part1 %})
* Part 2: [Introduction to Lucene 7 OpenNLP - Part 2]({% post_url 2018-10-01-introduction-to-lucene-opennlp-part2 %})
* Part 3: [Introduction to Lucene 7 OpenNLP - Part 3]({% post_url 2018-10-06-introduction-to-lucene-opennlp-part3 %})
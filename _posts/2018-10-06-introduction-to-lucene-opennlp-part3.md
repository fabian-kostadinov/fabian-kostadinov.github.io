---
layout: post
title: Introduction to Lucene 7 OpenNLP - Part 3
comments: true
tags: [lucene, opennlp]
---
Now that we can do [searching on indexed part-of-speech tags]({% post_url 2018-10-01-introduction-to-lucene-opennlp-part2 %}) what's still missing is a way to introduce an order of search terms. Remember: All POS tags in our query are simply **OR**ed together. So, how an we achieve this?<span class="more"></span>
Fortunately, this time the answer comes easily. We can use [PhraseQuery](https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/search/PhraseQuery.html). According to the official docs a _PhraseQuery_ is...

> A Query that matches documents containing a particular sequence of terms.

Phrase queries are useful if we want to search for a sequence of POS tags such as "DT JJ NN" (matching texts like "the big tree", "a remarkable man" etc.). To make things even better there is even a [query builder](https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/util/QueryBuilder.html) for this purpose:
```java
import org.apache.lucene.util.QueryBuilder;

// Some code

String fieldName = "body"; // or any other fieldName matching the indexed data
Analyzer analyzer = ...; // For example: new OpenNLPAnalyzer();
QueryBuilder builder = new QueryBuilder(analyzer);
Query query = builder.createPhraseQuery(fieldName, "DT JJ NN");
```
Executing the query will no longer yield documents with determiners, adjectives and nouns all mixed together, but instead yield only those indexed docs where this sequence of POS tags occurrs in the exact given order.

Cool stuff. What could we want more? Well, let's assume that we want to extend our search. Rather than searching only for the string "DT JJ NN" we also want to find strings "DT NN NN" (such as "a child genius").    
There are different ways to achieve this. One way would be to build two phrase queries, then build a boolean query and use an OR operator. Expressed in proper Lucene syntax, this would equate to:
```
+(body:"DT JJ NN" body:"DT NN NN")
```
_body_ refers to the field to search on. The two phrase queries in the bracket are **OR**ed together, wo either of them (or both) can match. In order to prevent that none of them matches yet (poorly matching) results are returend, we use a plus operator in front of the whole expression. This indicates that the final query MUST produce a match in its totality.
```java
QueryBuilder builder1 = new QueryBuilder(analyzer);
Query phraseQuery1 = builder1.createPhraseQuery(fieldName, "DT JJ NN");
Query phraseQuery2 = builder1.createPhraseQuery(fieldName, "DT NN NN");

BooleanQuery tmp = new BooleanQuery.Builder()
        .add(phraseQuery1, BooleanClause.Occur.SHOULD)
        .add(phraseQuery2, BooleanClause.Occur.SHOULD)
        .build();

// At this time both phrase queries are only ORed together. They SHOULD match,
// but this does not mean any of them MUST match. By using another combining
// boolean query with a MUST clause we guarantee that at least one of them
// matches.
BooleanQuery finalQuery = new BooleanQuery.Builder().add(tmp, BooleanClause.Occur.MUST).build();
```
Yet another, perhaps more concise way of achieving the same result would be to use a [MultiPhraseQuery](https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/search/MultiPhraseQuery.html). A _MultiPhraseQuery_ is...

> A generalized version of PhraseQuery, with the possibility of adding more than one term at the same position that are treated as a disjunction (OR).

Note that when using phrase queries and boolean queries we can rely on Lucene's [QueryParser](http://lucene.apache.org/core/7_4_0/queryparser/index.html?overview-summary.html) class. To my knowledge, no query parser exists for multi phrase queries in Lucene 7.4. We'll have to create our query object by instantiating query terms ourselves.
```java
MultiPhraseQuery query = new MultiPhraseQuery.Builder()
        .add(new Term(fieldName, "DT"))
        .add(new Term[] {new Term(fieldName, "JJ"), new Term(fieldName, "NN")})
        .add(new Term(fieldName, "NN"))
        .build();
```
Printing this multi phrase query to the console we see that it actually exactly produces the same query string as we had constructed above using boolean queries:
```
+(body:"DT JJ NN" body:"DT NN NN")
```
This brought me to yet another idea. Wouldn't it be possible to use regular expressions in order to build even more complicated query expressions? Something like this: "DT ((JJ)|(NN))+ NN". After a little research I indeed found out that in Lucene 7 there is a class [RegexpQuery](https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/search/RegexpQuery.html) that supports building queries following the syntax described in class [RegExp](https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/util/automaton/RegExp.html). Great, I thought, and implemented this:
```java
String searchRegex = "DT JJ NN";
Query query = new RegexpQuery(new Term(fieldName, searchRegex));
```
And the result? Nothing. This query actually returns 0 search results. At first, I was of course puzzled. It took me some time to realize that RegexpQuery is a subclass of the more general [AutomatonQuery](https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/search/AutomatonQuery.html) class. And AutomatonQuery is...

> A Query that will match terms against a finite-state machine.

You have to read this sentence very carefully. See my mistake? AutomatonQuery, and therefore also RegexpQuery, operates on individual indexed _terms_ - not on _sequences_ of terms like phrase queries do. Automaton queries and regular expression queries in Lucene can match a text like "DT JJ NN" only if they have been indexed not as separate terms but as a single term. WhitespaceAnalyzer, StandardAnalyzer and our own custom OpenNLPAnalyzer class all tokenize input texts relying on whitespaces (plus some more logic). They all split an input sequence of "the lazy dogs" into three tokens [the][lazy][dogs] based on which our OpenNLPAnalyzer adds the three POS tag tokens [DT][JJ][NN] again as three separate POS tags. In order to make use of RegexpQuery we'd however need [DT JJ NN] as a single token. Now, this can certainly be done during indexing time, but it would represent a new strategy of how to index terms. Whether or not a change of our indexing strategy is worth it of course depends on our use case. Furthermore, in general one should be careful when using regular expressions in queries. Regular expressions can be dangerous and certain types of regexes can actually bring our server down. From the [official Elasticsearch reference guide](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html):

> Regular expressions are dangerous because it’s easy to accidentally create an innocuous looking one that requires an exponential number of internal determinized automaton states (and corresponding RAM and CPU) for Lucene to execute.

## Other articles

### Previous
* Part 1: [Introduction to Lucene 7 OpenNLP - Part 1]({% post_url 2018-09-08-introduction-to-lucene-opennlp-part1 %})
* Part 2: [Introduction to Lucene 7 OpenNLP - Part 2]({% post_url 2018-10-01-introduction-to-lucene-opennlp-part2 %})

### Further material

* [Somewhat outdated article on automata in Lucene 4](https://opensourceconnections.com/blog/2013/02/21/lucene-4-finite-state-automaton-in-10-minutes-intro-tutorial/)
* [Another outdated article on multi phrase and other more advanced queries](http://blog.mikemccandless.com/2012/04/lucenes-tokenstreams-are-actually.html)
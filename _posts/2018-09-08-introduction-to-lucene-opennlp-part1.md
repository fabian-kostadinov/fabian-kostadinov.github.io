---
layout: post
title: Introduction to Lucene 7 OpenNLP - Part 1
comments: true
tags: [lucene, opennlp]
---
I've carried the idea to use OpenNLP to do part-of-speech tagging and index the POS tags with Lucene around with me for quite some time. Turns out Lucene 7 comes shipped with support for OpenNLP. Of course I had to try it out.<span class="more"></span>
Before starting, I highly recommend carefully reading through this official documentation of the Lucene analysis package: [https://lucene.apache.org/core/7_4_0/core/org/apache/lucene/analysis/package-summary.html]().

First, I created a new Java project in my IDE and added these two dependencies to my Maven's pom.xml:
```xml
<dependency>
  <groupId>org.apache.lucene</groupId>
  <artifactId>lucene-core</artifactId>
  <version>7.4.0</version>
</dependency>
<dependency>
  <groupId>org.apache.lucene</groupId>
  <artifactId>lucene-analyzers-opennlp</artifactId>
  <version>7.4.0</version>
</dependency>
```
Lucene uses a separate library just for the OpenNLP integration called _lucene-analyzers-opennlp_.

Let's briefly review the default way how to index a document using a StandardAnalyzer. According to the official documentation:

> An Analyzer is responsible for supplying a TokenStream which can be consumed by the indexing and searching processes.

And further below:

> The Analyzer is a factory for analysis chains. Analyzers don't process text, Analyzers construct CharFilters, Tokenizers, and/or TokenFilters that process text.

One point to remember is that if a document contains several fields that should be added to the index, by default an analyzer is applied to all indexed fields.   
So, here's the standard way of indexing documents in Lucene. 

```java
// Step 0: This is the text to be analyzed, i.e. indexed.
final String text = "The quick brown fox jumped over the lazy dogs.";

// Step 1: Create a directory, we'll use an in-memory directory for this purpose.
Directory memoryIndex = new RAMDirectory();

// Step 2: Create a new analyzer. StandardAnalyzer is very powerful and is sufficient for most cases.
// However, with OpenNLP we will implement our own analyzer further below.
StandardAnalyzer analyzer = new StandardAnalyzer();

// Step 3: Create an index writer config object, it takes an analyzer as its parameter.
IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);

// Step 4: Create the actual index writer giving it the directory object and the index writer config object.
IndexWriter writer = new IndexWriter(memoryIndex, indexWriterConfig);

// Step 5: We can now create new documents, and add some text fields to be indexed.
Document document = new Document();
document.add(new TextField("text", body, Field.Store.YES));

// Step 6: This line will effectively invoke the indexing process and send the new document to the writer.
writer.addDocument(document);

// Step 7: The document was indexed, and we can now close the writer.
writer.close();
```
After the indexing phase is over, we could now trigger queries on the indexed documents. A QueryParser also needs to apply the same type of analyzer to the query, otherwise it would not be possible to match the parsed query to the indexed documents. More details on querying will be covered in a later article.  

The analysis process converts text into indexable and thus searchable tokens. This works in several phases or layers.

1. First, a reader object reads a stream of individual characters. At this level, a Lucene _CharFilter_ could be used. A good example would be to drop all non-printable characters occurring in an input stream.
2. Next, a _Tokenizer_ assembles characters until it recognizes a token. Hence it returns a token stream.
3. Finally, a _TokenFilter_ can be applied to discard or modify tokens. Imagine that for anonymization you needed to replace all person names in a strem of tokens with a unique random string, this would probably be a good application for a token filter. Note that behind the scenes _TokenFilter_ actually extends _Tokenizer_.

Whereas using char filters and token filters is optional, using a tokenizer is not.

Next concept that we need to understand are attributes. The best explanation what Lucene's attributes are I found in [this tutorial](https://citrine.io/2015/02/15/building-a-custom-analyzer-in-lucene/).

> Lucene uses attributes to store information about a single token. For this tokenizer, the only attribute that we are going to use is the CharTermAttribute, which can store the text for the token that is generated. Other types of attributes exist (see interfaces and classes derived from org.apache.lucene.util.Attribute); we will use some of these other attributes when we build our custom token filter. It is important that you register attributes, whatever their type, using the addAttribute() function.

What sort of token information could one be possibly interested in to store? Well, there's plenty.

- The start and end character position or the token position of the token in the sentence or text. Welcome _OffsetAttribute_ and _PositionIncrementAttribute_.
- The token string (i.e. the word) itself. Use _CharTermAttribute_.
- Whether the token is a word, or perhaps only a punctuation. _TypeAttribute_ is your choice.
- Timestamp when it was processed or indexed. No library class available, build your own!

We wanted to do part-of-speech tagging, remember? So, yet another attribute we might be interested in is to store the POS information on the particular token. We have two options: Build our own POS attribute or, my preferred choice for the rest of the tutorial, reuse the existing _TypeAttribute_.

In order to do part-of-speech tagging we will use a token filter that for each token read from the token stream adds an additional type to the indexed data containing the part-of-speech tag for that token or word. For example, take this input sentence: _The quick brown fox jumped over the lazy dogs._ A tokenizer splits the sentence into tokens:

```
[The][quick][brown][fox][jumped][over][the][lazy][dogs][.]
```

(To keep the example simple, I will omit lower-casing, stopword-removal, stemming/lemmatization and other typical text preprocessing steps here.) In a second step, for each token consumed a POS token filter adds a part-of-speech tag as type attribute to the indexed data:

```
[DT][JJ][JJ][NN][VBD][IN][DT][JJ][NNS][.]
```

OpenNLP uses POS labels from the [Penn Treebank II tag set](https://www.clips.uantwerpen.be/pages/mbsp-tags), for example:

|Token   |Treebank POS tag|Description      |
|--------|----------------|-----------------|
|The     |DT              |Determiner       |
|quick   |JJ              |Adjective        |
|brown   |JJ              |Adjective        |
|fox     |NN              |Noun (singular)  |
|jumped  |VBD             |Verb (past-tense)|
|over    |IN              |Preposition      |
|the     |DT              |Determiner       |
|lazy    |JJ              |Adjective        |
|dogs    |NNS             |Nound (plural)   |
|.       |.               |Punctuation      |

In a later article, we'll see how we can search on these POS tags. For now I will focus on how we can achieve the above result.

Lucene's OpenNLP library contains only the basic building bricks for building your own analyzer, it does not contain a proper analyzer itself. For this reason, I created my own extending Lucene's abstract Analyzer class.
```java
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.TokenFilter;

public class OpenNLPAnalyzer extends Analyzer {
    protected TokenStreamComponents createComponents(String fieldName) {
        Tokenizer tokenizer = ...; // TODO
        TokenFilter tokenFilter = ...; // TODO
        return new TokenStreamComponents(tokenizer, tokenFilter);
    }
}
```
As mentioned above, the Analyzer's task is to create a token stream, but not to execute it (just yet). Creation is done in the _createComponents_ method. In case we want to apply additional filter to the token stream, the class _TokenStreamComponents_ can be used. The constructor of _TokenStreamComponents_ takes two arguments: a tokenizer and a token filter (which, as said above, happens to be of type _Tokenizer_ too). Note that in many cases it's not necessary to make use of the method argument _String fieldName_.

First thing we need is a Tokenizer. Lucene OpenNLP library has a class OpenNLPTokenizer:
```java
package org.apache.lucene.analysis.opennlp;

public class OpenNLPTokenizer extends SegmentingTokenizerBase /* which itself extends Tokenizer */ {
    public OpenNLPTokenizer(AttributeFactory factory, NLPSentenceDetectorOp sentenceOp, NLPTokenizerOp tokenizerOp) throws IOException {
        // ....
    }
}
```
The constructor takes three arguents:

- _AttributeFactory_: An attribute factory is simply a factory that creates instances of _Attribute_. Lucene has a very nice default implementation that is sufficient in most cases _AttributeFactory.DEFAULT_ATTRIBUTE_FACTORY_. A Lucene attribute consists of an interface and an implementing class with the suffix 'Impl'. The default attribute factory uses reflection to find the proper implementing class for an attribute interface. The convention is: For an interface _FooAttribute extends Attribute_ there must exist an implementation class _FooAttributeImpl extends AttributeImpl implements FooAttribute_. You will have to write your own attribute factory only in case you plan to not follow this naming convention for any reason.
- _NLPSentenceDetectorOp_: This is a class performing sentence detection provided by Lucene OpenNLP library.
- _NLPTokenizerOp_: This is a class performing tokenization provided by Lucene OpenNLP library.

In order to implement all NLP*Op classes we can use OpenNLPOpsFactory's static create methods. These methods load the corresponding [language-specific OpenNLP model files](https://opennlp.apache.org/models.html) (see also [here](http://opennlp.sourceforge.net/models-1.5/)) using a resource loader.
```java
import opennlp.tools.tokenize.TokenizerModel;
import opennlp.tools.sentdetect.SentenceModel;
import org.apache.lucene.analysis.opennlp.tools.NLPSentenceDetectorOp;
import org.apache.lucene.analysis.opennlp.tools.NLPTokenizerOp;
import org.apache.lucene.analysis.opennlp.tools.OpenNLPOpsFactory;
import org.apache.lucene.analysis.opennlp.OpenNLPTokenizer;
import org.apache.lucene.analysis.util.ClasspathResourceLoader;
import org.apache.lucene.analysis.util.ResourceLoader;
import org.apache.lucene.util.AttributeFactory;

// Some code...

// Get the current resource loader
ResourceLoader resourceLoader = new ClasspathResourceLoader(ClassLoader.getSystemClassLoader());

// Load OpenNLP's tokenizer model file using the resource loader and the OpenNLPOpsFactory
TokenizerModel tokenizerModel = OpenNLPOpsFactory.getTokenizerModel("en-token.bin", resourceLoader);

// Instantiate the NLPTokenizerOp using the model
NLPTokenizerOp tokenizerOp = new NLPTokenizerOp(tokenizerModel);

// Load OpenNLP's sentence detection model file using the resource loader and the OpenNLPOpsFactory
SentenceModel sentenceModel = OpenNLPOpsFactory.getSentenceModel("en-sent.bin", resourceLoader);

// Instantiate the NLPSentenceDetectorOp using the model
NLPSentenceDetectorOp sentenceDetectorOp = new NLPSentenceDetectorOp(sentenceModel);

// Instantiate the OpenNLPTokenizer
Tokenizer tokenizer = new OpenNLPTokenizer(AttributeFactory.DEFAULT_ATTRIBUTE_FACTORY, sentenceDetectorOp, tokenizerOp);
```
Awesome, we have a tokenizer now that can detect sentences and create word tokens. Next we want to create our POS token filter. After what we've seen It's pretty straight forward.
```java
import opennlp.tools.postag.POSModel;
import org.apache.lucene.analysis.TokenFilter;
import org.apache.lucene.analysis.opennlp.OpenNLPPOSFilter;
import org.apache.lucene.analysis.opennlp.tools.NLPPOSTaggerOp;

// Some code...

// Load OpenNLP's POS model file using the resource loader and the OpenNLPOpsFactory. Consult
// the OpenNLP documentation for the difference between maxent and perceptron model.
POSModel posModel = OpenNLPOpsFactory.getPOSTaggerModel("en-pos-maxent.bin", resourceLoader);

// Instantiate the NLPPOSTaggerOp using the model
NLPPOSTaggerOp posTaggerOp = new NLPPOSTaggerOp(posModel);

// Instantiate the token filter using the OpenNLPTokenizer and the NLPPOSTaggerOp instances
TokenFilter filter = new OpenNLPPOSFilter(tokenizer, posTaggerOp);
```
Nice! We can put everything together now.
```java
package com.example;

import opennlp.tools.postag.POSModel;
import opennlp.tools.sentdetect.SentenceModel;
import opennlp.tools.tokenize.TokenizerModel;
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenFilter;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.opennlp.OpenNLPPOSFilter;
import org.apache.lucene.analysis.opennlp.OpenNLPTokenizer;
import org.apache.lucene.analysis.opennlp.tools.NLPPOSTaggerOp;
import org.apache.lucene.analysis.opennlp.tools.NLPSentenceDetectorOp;
import org.apache.lucene.analysis.opennlp.tools.NLPTokenizerOp;
import org.apache.lucene.analysis.opennlp.tools.OpenNLPOpsFactory;
import org.apache.lucene.analysis.util.ClasspathResourceLoader;
import org.apache.lucene.analysis.util.ResourceLoader;
import org.apache.lucene.util.AttributeFactory;

import java.io.IOException;

public class OpenNLPAnalyzer extends Analyzer {
    protected TokenStreamComponents createComponents(String fieldName) {
        try {

            ResourceLoader resourceLoader = new ClasspathResourceLoader(ClassLoader.getSystemClassLoader());

            TokenizerModel tokenizerModel = OpenNLPOpsFactory.getTokenizerModel("en-token.bin", resourceLoader);
            NLPTokenizerOp tokenizerOp = new NLPTokenizerOp(tokenizerModel);

            SentenceModel sentenceModel = OpenNLPOpsFactory.getSentenceModel("en-sent.bin", resourceLoader);
            NLPSentenceDetectorOp sentenceDetectorOp = new NLPSentenceDetectorOp(sentenceModel);

            Tokenizer source = new OpenNLPTokenizer(
                    AttributeFactory.DEFAULT_ATTRIBUTE_FACTORY, sentenceDetectorOp, tokenizerOp);

            POSModel posModel = OpenNLPOpsFactory.getPOSTaggerModel("en-pos-maxent.bin", resourceLoader);
            NLPPOSTaggerOp posTaggerOp = new NLPPOSTaggerOp(posModel);

            TokenFilter filter = new OpenNLPPOSFilter(source, posTaggerOp);

            return new TokenStreamComponents(source, filter);
        }
        catch (IOException e) {
            // Do something...
        }
    }
}
```
This is it, we've created our own (POS) analyzer for Lucene. Let's see it in action. Note that our analyzer writes POS tags into the token's TypeAttribute.
```java
import com.example.OpenNLPanalyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.analysis.tokenattributes.TypeAttribute;

// Some code...

final String text = "The quick brown fox jumped over the lazy dogs.";

OpenNLPAnalyzer analyzer = new OpenNLPAnalyzer();

// We're using a string reader to return a token stream. This allows us to observe the tokens
// while they are being processed by our analyzer.
TokenStream stream = analyzer.tokenStream("field", new StringReader(text));

// CharTermAttribute will contain the actual token word
CharTermAttribute termAtt = stream.addAttribute(CharTermAttribute.class);

// TypeAttribute will contain the OpenNLP POS (Treebank II) tag
TypeAttribute typeAtt = stream.addAttribute(TypeAttribute.class);

try {
    stream.reset();

    // Print all tokens until stream is exhausted
    while (stream.incrementToken()) {
        System.out.println(termAtt.toString() + ": " + typeAtt.type());
    }

    stream.end();

} finally {
    stream.close();
}
```
Printed output:
```
The: DT
quick: JJ
brown: JJ
fox: NN
jumped: VBD
over: IN
the: DT
lazy: JJ
dogs: NNS
.: .
```
In part 2 of the tutorial we'll have a look how to search on the POS tags.

----
## Other articles

- [Exploring SolR's OpenNLP integrations](https://opensourceconnections.com/blog/2018/08/06/intro_solr_nlp_integrations/)
- [Building a custom analyzer, tokenizer and token filter in Lucene](https://citrine.io/2015/02/15/building-a-custom-analyzer-in-lucene/)
- [Full Text Search of Dialogues with Apache Lucene: A Tutorial](https://www.toptal.com/database/full-text-search-of-dialogues-with-apache-lucene)
- [Lucene's TokenStreams are actually graphs!](http://blog.mikemccandless.com/2012/04/lucenes-tokenstreams-are-actually.html)
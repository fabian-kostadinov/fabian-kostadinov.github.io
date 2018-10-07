---
layout: post
title: Introduction to Lucene 7 OpenNLP - Part 2
comments: true
tags: [lucene, opennlp]
---
In my [previous post]({% post_url 2018-09-08-introduction-to-lucene-opennlp-part1 %}) I promised I'd describe how to perform searches on indexed part-of-speech data with Lucene 7 and OpenNLP. Let's have a look. (Thanks to [Koji](https://stackoverflow.com/users/10277631/koji) on this one!)<span class="more"></span>
We have already seen how to create an index and then add some data.
```java
Directory index = new RAMDirectory();
OpenNLPAnalyzer analyzer = new OpenNLPAnalyzer();
IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);
IndexWriter writer = new IndexWriter(index, indexWriterConfig);

final String fieldName = "body";

Document document1 = new Document();
// POS tags: [DT][JJ][JJ][NN][VBD][IN][DT][JJ][NNS][.]
document.add(new TextField(fieldName, "The quick brown fox jumped over the lazy dogs.", Field.Store.YES));
writer.addDocument(document1);

Document document2 = new Document();
// POS tags: [VB][PRP][TO][PRP][,][UH][.]
document2.add(new TextField(fieldName, "Give it to me, baby!", Field.Store.YES));
writer.addDocument(document2);

Document document3 = new Document();
// POS tags: [NNP][NNP][VBZ][DT][JJ][NNS][.]
// Note that the token [Mr.] - including the dot - results in [NNP].
document3.add(new TextField(fieldName, "Mr. Robot is a great TV-series.", Field.Store.YES));
writer.addDocument(document3);

Document document4 = new Document();
// POS tags: [VB][PRP][TO][PRP][,][NNP][.]
document4.add(new TextField(fieldName, "Give them to us, Dalton!", Field.Store.YES));
writer.addDocument(document4);

writer.close();
```
The exact field name used - here it is _body_ - does not really matter too much, you can use any name you want. But you must make sure that during searching further below you again use the same field name! Furthermore, it is generally recommended to use the same analyzer as you've used to index your data with. In our case this is OpenNLPAnalyzer. This is however not a strict must. In a real search application there are reasons why you may want to create different analyzers for indexing data and for parsing queries.
```java
// Searching for documents containing both (at least one) adjective or (at least one) noun
final String searchPhrase = "JJ NN";

// The maximum number of top n returned results
final int topN = 10;
DirectoryReader reader = DirectoryReader.open(index);
IndexSearcher searcher = new IndexSearcher(reader);

// fieldName was specified above to be string "body"
QueryParser parser = new QueryParser(fieldName, new OpenNLPAnalyzer());
Query query = parser.parse(searchPhrase);
System.out.println(query);

TopDocs topDocs = searcher.search(query, topN);
System.out.printf("%s => %d hits\n", searchPhrase, topDocs.totalHits);
for(ScoreDoc scoreDoc: topDocs.scoreDocs){
    Document doc = searcher.doc(scoreDoc.doc);
    System.out.printf("\t%s\n", doc.get(field));
}
```
Here is the output:
```java
body:JJ body:NN
JJ NN => 0 hits
```
Our search yielded no results! How is this possible? Well. This is where things unfortunately get a bit complicated. There's something I haven't told you in the [previous post]({% post_url 2018-09-08-introduction-to-lucene-opennlp-part1 %}). [Elizabeth Haubert mentions it in her blog post](https://opensourceconnections.com/blog/2018/08/06/intro_solr_nlp_integrations/):

> Since Lucene does not yet index token types, in order to make that information available to queries, it is necessary to push the type either to a payload or as a synonym token using either TypeAsPayloadFilterFactory or TypeAsSynonymFilterFactory.

If you are like me you didn't understand this piece of information while reading it the first time and simply took a whole-hearted decision to ignore it. What does this mean? Well, it means exactly what it states: For any reason known only to the developers of the Lucene library the _type_ token is not indexed by default. And of course this is exactly what OpenNLP uses behind the scenes. In class _OpenNLPPOSFilter_ you will find these two lines:
```java
private final TypeAttribute typeAtt = (TypeAttribute)this.addAttribute(TypeAttribute.class);

// And further below while parsing tokens:
this.typeAtt.setType(this.tags[this.tokenNum++]);
```
_OpenNLPPOSFilter_ that we were using in our _OpenNLPAnalyzer_ class uses internally a _TypeAttribute_. And, as we've just learned, type attributes are not indexed by Lucene. So what to do? Well, the answer is given above. Use either _TypeAsPayloadFilterFactory_ or _TypeAsSynonymFilterFactory_ instead. Now a word of caution: Trying to use the former will bring you in hell's kitchen. No documentation paired up with outdated tutorials due to fundamental design changes of payloads between major Lucene versions will make your hair turn white and your teeth fall out. So, use the latter. You have been warned.

Ok, so how can we do it? Only a small change in our _OpenNLPAnalyzer_ class is needed. We simply need to wrap our _OpenNLPPOSFilter_ into a _TypeAsSynonymFilter_ a done in the return statement. The class can be found in lucene-analyzers-common library.
```java
package com.example;

import opennlp.tools.postag.POSModel;
import opennlp.tools.sentdetect.SentenceModel;
import opennlp.tools.tokenize.TokenizerModel;
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenFilter;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.miscellaneous.TypeAsSynonymFilter;
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

            return new TokenStreamComponents(source, new TypeAsSynonymFilter(filter));
        }
        catch (IOException e) {
            // Do something...
        }
    }
}
```
_TypeAsSynonymFilter_ performs a little trick behind the scenes. It adds the type information as a synonym to the term attribute. This means that going forward an adjective like [quick] and the POS type token [JJ] will be treated as synonyms.

Let's execute the same search as before. This time the following results are returned:
```java
body:JJ body:NN
JJ NN => 2 hits
	The quick brown fox jumped over the lazy baby.
	Mr. Robot is a great TV-series.
```
Congratulations, we found two indexed documents containing both an adjective and a noun. Since search terms are **OR**ed together the order of our POS tags in the query does not matter. "JJ NN" will yield the same results as "NN JJ". Furthermore, since words and their POS tags are treated as synonyms we can also mix them in our query. Note however that the adjective "brown" might be treated synonymously with the POS tag "JJ", but the opposite is not true!

Maybe if I find the time I'll write another piece how introduce order on our query terms. Searching for "JJ JJ NN" should not simply OR them together, but should preserve the sequence of query terms.

----

## Other articles

### Previous
* Part 1: [Introduction to Lucene 7 OpenNLP - Part 1]({% post_url 2018-09-08-introduction-to-lucene-opennlp-part1 %})

### Next
* Part 3: [Introduction to Lucene 7 OpenNLP - Part 3]({% post_url 2018-10-06-introduction-to-lucene-opennlp-part3 %})
* Part 4: [Introduction to Lucene 7 OpenNLP - Part 4]({% post_url 2018-10-07-introduction-to-lucene-opennlp-part4 %})
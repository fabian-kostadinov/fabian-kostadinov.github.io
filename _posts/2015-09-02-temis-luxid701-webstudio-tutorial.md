---
layout: post
title: Temis Luxid 7.0.1 Webstudio Tutorial
comments: true
tags: [temis, luxid, webstudio]
---
As I was not able to find any tutorials on the web on how to use Temis Luxid 7.0.1 Webstudio, I simply decided to write my own. Luxid Webstudio is a tool that is intended for different use cases. One thing it does very well is to assist a taxonomy expert to build a new taxonomy or enrich an existing one with new terms. Furthermore, once a taxonomy is created it can be "plugged in" to the STF skill cartridge, which then is able to extract all the taxonomy terms from documents. By exporting this customized skill cartridge from Webstudio, you can simply deploy it to a dedicated annotation server running in your production environment. Some of Webstudio's functionality overlaps with the Eclipse-RCP based Luxid 7.0.1 _Annotation Workbench_, however Webstudio is simply more comfortable to use. Only in some cases it is necessary to switch to Annotation Workbench because it exposes even more functionality to the user than Webstudio does.<span class="more"></span>

The work performed with Webstudio is organized in projects, and each project is stored inside the same MySQL database. Other Luxid tools like Annotation Workbench or Rich Annotation Pad (which is nothing else than a subset of Annotation Workbench) access the same database and therefore you can work on the same data and projects although the tools at their surface look quite different.

In this tutorial you'll learn...

- how to import an existing taxonomy,
- enrich it with new terms,
- and finally export and deploy it to a production server.

__Step 1__  
Log in to Luxid 7.0.1 Webstudio. By default, Webstudio is available at http://dev-server:8060/LuxidStudio, where _dev-server_ of course refers to the server where Luxid Content Enrichment Studio is stored.

__Step 2__  
Create a new project, chose any name you want. In the background, projects are actually stored in a MySQL database.

__Step 3__  
Usually, when working with Webstudio, there are two steps to be taken.

1. Install the skill cartridge you want to use in your project.
2. Upload sample documents you want to work with.

__Step 4__  
In this tutorial we'll assume that you will use the STF (Suggested Term Finder) skill cartridge. The STF skill cartridge (together with the IncreaseSTFPrecision cartridge) is already installed by default in every new Luxid 7.0.1 Webstudio project. When you go to _> Project > Configure... > Annotation_ you actually see the annotation plan with all annotators contained therein.  

The _STF_ annotator is aims at finding terms already contained in a taxonomy in a corpus of documents.

The _IncreaseSTFPrecision_ annotator takes the STF's output and removes certain extracted terms from it. Assume there is a taxonomy concerning animals. This taxonomy includes the terms "cat" and "dog". Further assume that there is a document containing the sentence "It's raining cats and dogs." Then surely "raining" indicates that semantically this sentence is referring to a weather phenomenon - not to animals. It's a metaphor, in other words. We want to prevent "cats" and "dogs" being extracted when they appear together with the word "raining". Every term in our taxonomy has two special attributes that can be used in such situations: _Forbidden Spans_ and _Context-Based Stop List_. In this case we could use the latter and add "raining" to our list of stop words. This means that whenever the word "raining" appears in the same context (sentence) as "cats" or "dogs" then these terms will actually not be extracted as animals. This is the IncreaseSTFPrecision's task.

Once we will be done with building our taxonomy, we can either export the taxonomy in a [SKOS format](http://www.w3.org/2004/02/skos/) or build and deploy a new skill cartridge that combines the taxonomy, the STF and IncreaseSTFPrecision skill cartridges and deploy them to our backend annotation server.

If you did not want to use the STF (and IncreaseSTFPrecision) skill cartridge but for example the TM360 or WPS (Wikipedia Synonyms) cartridge, then you'd have to uninstall the default cartridges first and then install the TM360 or WPS. (As a side note, I ran into problems when trying to work with the TM360 inside Luxid 7.0.1 Webstudio. My hope is that the already announced version 7.1 will solve these.)

__Step 5__  
There is a bug in Luxid 7.0.1 Webstudio which causes problems when searching for taxonomy terms in documents. To circumvent it, go to _> Project > Configure... > Languages_ and then make sure that the language _-- Unspecified --_ is selected.

__Step 6__  
Now that we've installed our skill cartridge, we must upload a corpus of sample documents. I don't recommend starting with too many documents in the beginning. Be aware that the import process may take some time as each document is first converted from its original format into an internal HTML format and further pre-annotated with certain metadata. The converted documents are then stored inside the MySQL database and all processing is performed on this internal data. This is however true only for the Content Enrichment Studio (CES) tools. The annotation server that is included in the CES tools operates on the documents in the MySQL database. The annotation server intended for productive use does not, one must either push documents to it through a REST API or tell it to pull documents from a certain location. In both cases it only processes the documents but does not store them anywhere.

By the way, in Webstudio deleting a document from the imported corpus is not possible, this is one of the features currently only available in the Annotation Workbench.  

Many more than only the following file formats are supported, however be aware that the quality of the extraction results may vary according to document types:

- Word (doc, docx)
- PDF
- Text
- HTML, XML
- Powerpoint (ppt, pptx)
- Excel (xls, xlsx)
- Many more

A few remarks:

- What is processed is _text content_ from a file. If the file does not contain text or only little then of course nothing can be extracted. I have seen Word and PDF files containing nothing but images (for example screenshots). Luxid is not an optical character recognition (OCR) system. It cannot process data from images, audio or video files. This should be self-evident but especially when you're dealing with data that was uploaded by users in an uncontrolled way to a shared repository such as Microsoft Sharepoint you will almost certainly encounter such documents.
- Documents that do not contain any text at all (for example .dll or .exe files) are simply ignored by Luxid.
- According to my experience extracting data from Excel files is much more difficult than from Word, PDF or Text files. Excel data is fundamentally table-based, it's sorted in rows and columns. This is very different from common text which is fundamentally a single long string of words and characters.
- When processing HTML or XML files it is recommended to use the _LuxidConverter_ skill cartridge first in any annotation plan. This does not refer to Webstudio but to the annotation plans installed in the backend annotation server.

So, let's upload our sample documents. Go to _> Project > Add Documents..._ and select the file(s) you want to upload. A nice feature, which is unfortunately not mentioned explicitly, is that you can upload a whole zip file containing several text files.

__Step 7__  
When working with the STF skill cartridge, most of the time we also work with a taxonomy. In case you already have an existing taxonomy that you want to enrich with new terms, you first need to import this taxonomy into Webstudio. On the left-hand side below the Luxid Webstudio logo, you'll find an empty _Thesaurus_. If you click on the icon with the three horizontal bars, you can select _Import Thesaurus..._. The taxonomy should usually be in a SKOS format, but a few different file formats can be imported in the Annotation Workbench. This is an exmample for a very simply taxonomy in the SKOS format consisting of only three terms: animal, which will not be extracted (<code>&lt;luxid:DoNotExtract&gt;true&lt;/luxid:DoNotExtract&gt;</code>) and a term "Dog" and "Cat" which are subterms of "Animal".

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<rdf:RDF
   xmlns:skos="http://www.w3.org/2004/02/skos/core#"
   xmlns:luxid="http://www.temis.com/luxid#"
   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
   xmlns:my-animals="http://www.example.com/animal-taxonomy#">

   <!-- Animal -->
   <rdf:Description rdf:about="http://www.example.com/animal-taxonomy#Animal">
      <rdf:type rdf:resource="http://www.w3.org/2004/02/skos/core#Concept"/>
      <luxid:DoNotExtract>true</luxid:DoNotExtract>
      <skos:prefLabel xml:lang="en">Animal</skos:prefLabel>
      <skos:prefLabel xml:lang="de">Tier</skos:prefLabel>
   </rdf:Description>

   <!-- Dog -->
   <rdf:Description rdf:about="http://www.example.com/animal-taxonomy#Dog">
      <rdf:type rdf:resource="http://www.w3.org/2004/02/skos/core#Concept"/>
      <skos:broader rdf:resource="http://www.example.com/animal-taxonomy#Animal"/>
      <skos:prefLabel xml:lang="en">Dog</skos:prefLabel>
      <skos:prefLabel xml:lang="fr">Chien</skos:prefLabel>
      <my-animals:database-id>1234</my-animals:database-id>
   </rdf:Description>

   <!-- Cat -->
   <rdf:Description rdf:about="http://www.example.com/animal-taxonomy#Cat">
      <rdf:type rdf:resource="http://www.w3.org/2004/02/skos/core#Concept"/>
      <skos:broader rdf:resource="http://www.example.com/animal-taxonomy#Cat"/>
      <skos:prefLabel xml:lang="en">Cat</skos:prefLabel>
      <skos:prefLabel xml:lang="de">Katze</skos:prefLabel>
      <my-animals:database-id>1235</my-animals:database-id>
   </rdf:Description>

</rdf:RDF>
{% endhighlight %}

Once imported, you can use the taxonomy in Webstudio.

There exists a nice, free taxonomy by the name of _STW (Standard-Thesaurus Wirtschaft)_ primarily containing economics terms published and maintained by the [ZBW German National Library of Economics](http://zbw.eu/en/) that you can use right out of the box. This taxonomy is in the RDF format which neatly integrates with SKOS. It can be found at [http://zbw.eu/stw/versions/latest/download/](http://zbw.eu/stw/versions/latest/download/).

There are different ways to interact with the taxonomy. A simple task is to add new terms anywhere in the hierarchy or delete existing ones by clicking on the plus and minus sign next to any term. A very clever feature is to receive suggestions for new terms not yet part of the taxonomy. We'll do this further below.

__Step 8__  
Open a term by clicking on it in the Thesaurus section. Every term in the taxonomy contains a few attributes or fields.

_General Attributes_

- _ID_: The ID of this term. This is not a database ID but rather a RDF-style ID, for example "http://www.example.com/animal-taxonomy" or "http://www.foo.com/cities/Mumbai".
- _Preferred Labels_: The main or preferred label of this term, for example "Mumbai". The STF skill cartridge will try to extract this text string from documents and indicate it has found a "Mumbai" term.
- _Alternative Labels_: Alternative labels of the term, e.g. "Bombay". The STF skill cartridge will try to extract this text string from documents also. If found it will however indicate a "Mumbai" term.
- _Hidden Labels_: Labels of the term that are explicitly not extracted.

_Extraction_

- _Do Not Extract_: Set to true if for any reason you don't want to extract this term.
- _Extraction Method_: _Fuzzy Matching_ uses a, well, fuzzy string matching approach to match a term. This may still extract terms with misspellings. _Same Form_ extracts terms that have an exact (case-sensitive or insensitive) match of characters.
- _Forbidden Spans_: Used by IncreaseSTFPrecision annotator.
- _Context-Based Stop List_: Used by IncreaseSTFPrecision annotator.

_Relationships_

- _Broader Concepts_: Concepts higher up in the taxonomy's hierarchy.
- _Narrower Concepts_: Concepts further below in the taxonomy's hierarchy.

_Notes_

- _Definition_: A field where you can enter a free text definition for this particular term or concept.
- _Scope note_: A field where you can enter additional notes concerning the scope of this particular term or concept.
- _Notation_: See [SKOS definition](http://www.w3.org/TR/skos-reference/#notations) for more details.

_Custom Attributes_

- _Custom attributes_: Attributes already contained in the taxonomy that have no meaning to Webstudio may be stored as custom attributes.

__Step 9__  
By clicking on the small eye icon ("Preview sentences potentially containing this concept") the corpus of text documents is searched for occurrences of this term. Depending on the corpus and taxonomy size this may take a moment. The results are displayed in the _Preview_ section. In this way you can check the quality of the extracted results and make corrections to your taxonomy.

Click on the _alternatives icon_ next to the eye icon to receive a list of synonym candidates for this term. This is very useful to find alternative labels or possibly new terms.

__Step 10__  
You can receive suggestions for new terms not yet contained in the taxonomy (and not being synonyms neither). Click on the _lamp icon_ next to the _Thesaurus_ label. In the suggestions panel you can see a ranked list of terms that are potentially relevant to you and are not yet part of the taxonomy. There are different sorts of ranking, and you can adapt them according to your need. Drag and drop suggested terms simply from this list to your taxonomy.

__Step 11__  
When you are done with creating/enriching the taxonomy, you can actually export again to a file. Click on the icon with the three little horizontal bars close to the term _Thesaurus_, then select _> Export Thesaurus..._.

__Step 12__  
We are ready to export our taxonomy & STF skill cartridge to deploy it on the production annotation server. Click on _> Projects > Export Annotation Plan..._. In the background Webstudio builds a _skill cartridge archive (.sca)_ file that is by default stored in your personal _Download_ folder. Every .sca file is actually nothing else than a zip file. If you want you can rename it to .zip and open it with WinZip or a similar tool. This .sca file is a fully self-contained annotation plan. Internally it contains:

1. its own copy of the taxonomy (in binary format),
2. its own copy of the STF skill annotator,
3. its own copy of the IncreaseSTFPrecision annotator.

Being fully-self contained means that on the production server it will not have any cross-dependencies to other skill cartridges already installed there. On the positive side .sca files can easily be deployed and undeployed to and from the production annotation server. On the negative side they may become very big. This is especially true when working with the TM360 skill cartridge.

__Step 13__  
Log in to the production annotation server. There is usually an admin web app available at http://prod-server:8061/Luxid7Admin. You'll need an administrator username and password. Once logged in you simply can select "Install Skill Cartridge" and select the .sca file that you've just exported.

__Step 14__  
Now that you've installed it, you can send documents to the server's REST API to be annotated. The following cURL command can also be found in the REST API guide distributed together with the backend annotation server. We'll assume that you just installed an annotation plan from a file named "MyAnimals.sca".

Example 1: Sending a simple text string to the server:
{% highlight console %}
curl -X POST -H "Content-Type: text/plain" -d "Hello cat and hello dog! What sort of animal are you?" http://prod-server:8091/temis/v1/annotation/annotate/MyAnimals
{% endhighlight %}

Example 2: Sending a PDF document to the server:
{% highlight console %}
curl -X POST -H "Content-Type: application/octet-stream" --data-binary "@C:\myDocument.pdf" http://prod-server:8091/temis/v1/annotation/annotate/MyAnimals
{% endhighlight %}

Depending on the document's contents you should receive a lengthy XML output in the LUX format. LUX is a XML format specification invented by Temis. The corresponding documentation is also distributed together with the REST API and the production annotation server. The LUX output contains:

- The raw text as it has been transformed from the original input document,
- a list of sentence begin and end positions,
- the extracted entities such as "/Thesaurus/Concept/Animal/Dog",
- their occurrences in the text,
- eventually further attributes for each extracted entity.

If you only receive minimal or no output at this stage, you'll probably need to understand more about the concept of _static mapping_. The concept is not very difficult to understand, however changing static mapping rules in Webstudio is unfortunately not possible, and is rather complicated in Annotation Workbench. The basic idea is that every term in the taxonomy's hierarchy is "bound" to a specific path inside the hierarchy, for example "/Thesaurus/Concept/Animal/Dog". If you built your taxonomy in Webstudio your taxonomy will probably not contain a term "Concept" and for example the term for "Dog" will be placed in "/Thesaurus/Animals/Dog". However, at the same time in _> Project > Configure... > Annotation_ in the section _Annotation Strategies_ the default settings are that all terms will be bound to "/Thesaurus/Concept" and not simply "/Thesaurus". There are only two possibilities here: Either you change this mapping inside Annotation Workbench manually, or you accept your fate, create a new root term "/Thesaurus/Concept" and move your whole taxonomy under that term. Of course you need again to export everything as a .sca file and re-import it to the production annotation server.

__Step 15__  
This output could now be processed further by a service client, for example to highlight all occurrences of animal names in an original text.
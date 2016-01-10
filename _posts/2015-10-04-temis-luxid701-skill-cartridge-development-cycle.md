---
layout: post
title: Temis Luxid 7.0.1 Skill Cartridge Development Cycle
comments: true
tags: [temis, luxid, development]
---
Skill cartridges built with Luxid 7 usually contain a mix of customized and standard software artefacts. These artefacts can be data artefacts such as tailored vocabularies or taxonomies, syntactic or similar rules to extract certain types of entities, or they can be a set of configuration files that parameterize the skill cartridge at hand. For this reason, skill cartridges must be treated as productive code and must therefore be subject to a build and deployment process as well as be checked into a version control system. The good news is that Temis has made it really easy to set up your own version of this process. The bad news is that at least in Luxid 7.0.1 there does not seem to exist any documentation on the corresponding tools.<span class="more"></span>

## Skill Cartridge Development Life-Cycle
The first thing to understand is the typical development-cycle of a skill cartridge.

1. The skill cartridge is shipped by the vendor as a zip file. The zip file contains both a "bare-bone" or empty skill cartridge plus possibly a file called _customization.zip_. This file again contains one or several default dictionaries that may eventually be loaded into the skill cartridge.

2. Content Enrichment Studio (CES) relies on projects kept inside a (MySQL) database. All of a project's contents - dictionaries, skill cartridges, sample documents etc - are kept inside the database - not in the file system! When you create a new project, you have to load all the different parts into the database, that is dictionary, skill cartridge and sample documents.

3. Usually, it's best to first load the knowledge into your project. As said above, skill cartridges like TM360 are shipped with a _customization.zip_ file containing default dictionaries. Simply load this file into your project.

4. Next, you load the empty skill cartridge (e.g. TM360.sca) into your project. At this stage, knowledge and skill cartridge are still separated from each other. Applying the skill cartridge on any documents would not return any dictionary results yet.

5. Hence, you synchronize your skill cartridge with the knowledge in the project. It is now charged, that is it can extract knowledge contained in the dictionaries from text.

6. You load your sample dictionaries into your project.

7. You measure the quality of the extraction results of the skill cartridge. If not yet satisfied, you improve the quality of the extraction output. You can also load additional dictionaries to your project (step 3), but afterwards need to synchronize the skill cartridge again (step 5).

8. When satisfied with the extraction results, you choose to export your charged skill cartridge from the database. A build process is initiated that creates a fully self-contained skill cartridge archive (.sca) file, which is nothing else than a zip file with a specific internal folder structure. You can rename the file to .zip and open it if you wish to do so.

9. At this stage you should actually add your skill cartridge to a version control system, or at least those parts that may have changed if the skill cartridge already existed as a CES project.

10. Once you have a .sca file you can simply upload it to your backend annotation server using either a REST API call (see the Luxid 7 REST Annotation API guide v1.1 for the details) or the admin web app.

As we'll see below there are tools to support you during this development-cycle.

Once a skill cartridge is installed in a backend annotation server, certain parts of the dictionaries may be still subject to manipulation with a text editor, whereas other parts are represented as binary files and cannot be manipulated anymore. In Luxid the parts that still can be manipulated are called _externalized_ knowledge or dictionaries. If you simply change externalized knowledge, not much will happen to the installed skill cartridge. After making a change you have to actively synchronize the skill cartridge with the externalized knowledge, in other words you have to execute the synchronization step. Some skill cartridges have their own synchronize procedures for this purpose that you can call through a REST API. Be aware that I have never tested this functionality, so I might be wrong in this regard.  

As described above, for skill cartridges not installed in the backend annotation server but as parts of CES projects, synchronization is triggered either from inside the CES tools or during the export process. For example in Webstudio go to _Project > Configure... > Annotation_, then press the _Synchronize_ button.

## Building skill cartridges
Basically, there are four ways of building your own skill cartridge in Luxid 7. You can

- use Webstudio,
- use Skill Cartridge Builder (SCB) and possibly Annotation Workbench (AWB),
- implement your own cartridges in Java or Groovy either as a simple script or as a complete skill cartridge, or
- build annotation plans in admin web app by configuration.     

### Using Webstudio
As pointed out above, if you build your skill cartridge in Webstudio then exporting your skill cartridge will usually contain at least a vocabulary, lexicon, taxonomy or similar plus the "bare-bone" skill cartridge like STF or TM360.  

I believe in Luxid 7.1 the deployment step with Webstudio has become even easier. You can actually deploy your skill cartridge directly from Webstudio to the backend annotation server with a single mouse click. This is cool if your backend annotation server does not run any mission critical system, it may be fatal otherwise. So be careful. Best to have different strictly separated environments for development, testing and production in place.

### Using Skill Cartridge Builder
In case you're working with SCB, then the process is actually similar. Just press the export button and your skill cartridge is built together with all its dependencies. Then upload the .sca file to your backend annotation server.

### Writing custom Java code
When you want to write your own skill cartridges in Java or Groovy you can either write so called "scripts" or build whole applications. A script is simply a single Java or Groovy class that extends a predefined annotator class and has no further dependencies (e.g. to other .jar files). It then provides several method stubs where you can fill in your code. The whole architecture is similar to writing Java servlets, however there are important differences. In Luxid 7.0.1 the corresponding Java API is unfortunately not very well documented, you'll have to specifically ask Temis for more information. For some functionality I wanted to try out, I received an exception message stating that this particular function was not implemented yet. I don't know about the status in Luxid 7.1, but I expect some advancements in this regard.  
Luxid 7.0.1 admin web app still contained an "install script" button that was apparently removed in version 7.1. The documentation guide 1.1 of the REST API does not mention this explicitly, but you actually can install and uninstall a script through this a REST API call:

{% highlight console %}
curl -T MyScript.java http://server:8091/temis/v1/annotation/scripts/
{% endhighlight %}

In case you want to install a machine-learned model (.klm file) instead of a script then the REST URL is slightly different:

{% highlight console %}
curl -T MyModel.klm http://server:8091/temis/v1/annotation/models/
{% endhighlight %}

If you want to write your own customized skill cartridges I highly recommend having a look at the code samples that are delivered together with every Luxid installation. In a default installation, these code samples can be found in <code>C:\Temis\Luxid7\IDE\doc</code>.

Writing scripts in many cases is not sufficient. Imagine that for any reason you want to open a database connection in your custom Java code. You'll need a bunch of third-party .jar files to do so plus possibly some configuration files etc. All of these must be packaged and deployed together with your .sca file. These dependencies _must_ be placed in an internal folder called <code>/plugins</code>, otherwise they can not be found from inside your Java code. Remember that every .sca is fundamentally just a zip file that contains a specific folder structure. With the tools I'll show you further below it is actually pretty easy to write an [Ant](http://ant.apache.org/) or [Maven](http://maven.apache.org/) script that automates the build process for this .sca file.

One problem with Webstudio, SCB or admin web app is that when you push the export button you end up with a prebuilt .sca file. This file can sometimes be several hundred MB, because it may contain many different lexica. You don't want to check in the whole file to version control but only those parts that actually have changed. As I've said above, every .sca file is nothing but a zip file. You can always unpack it to a folder in your file system which is part of your version control system. The version control system should then be able to see only the differences between the new and the old version for each file of the unpacked .sca.

### Building annotation plans
You can of course also build whole annotation plans in admin web app just by configuring existing annotators. An annotator in an annotation plan can be any combination of these four:

- a skill cartridge,
- a Java/Groovy script or customized skill cartridge,
- a machine-learning model plus Analytics/Analytics2/RTF cartridge,
- another annotation plan.

Once you export your annotation plan, again a .sca file is built. This file contains all dependencies inside. Of course such a file can potentially become very big, especially if your annotation plan relies on other annotation plans. (Just for the record: nope, I did not test whether you can build cyclic dependencies... .) The whole annotation plan is thus self-contained. You can import it to a backend annotation server like every regular skill cartridge.

## Windows build & deployment utilities
Assuming you really want to have full control over the build process in CES yourself you can use a set of Windows utilities available in <code>C:\Temis\Luxid7\IDE\bin\\</code> or <code>C:\Temis\Luxid7Studio\IDE\bin\\</code>. [The following information I received from a [discussion thread](http://community.temis.com/group/temis-extranet/forum/-/message_boards/message/152221) at the internal [Temis community forum](http://community.temis.com).]

- _scinstall.bat_: Installs a bare-bone skill cartridge in a CES project. Empty here means that the skill cartridge at this moment does not yet contain any vocabulary, taxonomy etc. These must be loaded using the <code>crimport.bat</code> tool.
- _crimport.bat_: Import knowledge into a Studio project, for example a vocabulary, taxonomy etc.
- _crassign.bat_: Assigns the skill cartridge to a Studio-project
- _crsynchro.bat_: Synchronize the still empty skill cartridge with the imported knowledge. This step "charges" the vocabulary or taxonomy into the skill cartridge.
- _crexportsca.bat_: Build and export the skill cartridge as a single .sca file to the file system.

In Luxid 7.1 the same functionality is actually accessible through a new REST API to the Studio annotation server. (I don't know whether 7.0.1 already provided such an API.) As with these Windows batch scripts I could not find any documentation for the Luxid 7.1 Studio REST API. If you have more documentation on both the CES 7.1 REST API or the Windows utilities, I'll definitely be interested.

----

# Luxid 7.0.1 export bug
Please note that in Luxid 7.0.1 there still exists a terrible bug. Whenever you try to export a configured annotation plan from admin web app while a document is currently being annotated using exactly this annotation plan, then this causes a corruption of the annotation plan. As a result subsequent attempts to annotate a document will fail and error messages will be returned - usually IOExceptions or sometimes XeldaExceptions. When you open the admin web app you will see that the annotator that was in use when the export button was pressed seemingly "disappeared" from the annotation plan. Yet, you will still see the skill cartridge being properly installed when calling the backend annotation server's REST API:

{% highlight console %}
curl http://prod-server:8091/temis/v1/annotation/cartridges.xml
{% endhighlight %}

Hence, there is a mismatch between what is displayed as a result to the REST API and what is listed in the admin web app. Restarting the server does not solve the problem. The only way to solve this issue is to actually reinstall exactly that skill cartridge that is missing in the admin web app (and then restart the server). I really hope this issue has been solved in Luxid 7.1. I know the Temis engineers were aware of this bug before rolling out Luxid 7.1, so I'm pretty positive that this was addressed although I did not check specifically so far.

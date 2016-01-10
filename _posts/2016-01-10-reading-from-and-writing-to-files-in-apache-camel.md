---
layout: post
title: Reading from and writing to files in Apache Camel
comments: true
tags: [apache camel]
---
I had assumed that reading from and writing to files in [Apache Camel v2.16.1](http://camel.apache.org/) should be a straight-forward thing to accomplish. Turns out I was wrong. It took me quite a while to figure out the correct syntax of the <code>from</code> and <code>to</code> commands.<span class="more"></span>

# Reading a single text file
Before we can use Apache Camel, we need to import it in our pom.xml Maven file:

{% highlight xml %}
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-stream</artifactId>
    <version>2.16.1</version>
</dependency>
{% endhighlight %}

There are various ways to read files in Apache Camel. If the files are in plain text format the <code>org.apache.camel.builder.RouteBuilder</code>'s <code>from</code> method is probably the best choice. The <code>from</code> method is overloaded:

{% highlight java %}
public RouteDefinition from(Endpoint... endpoints)
public RouteDefinition from(Endpoint endpoint)
public RouteDefinition from(String... uris)
public RouteDefinition from(String uri)
{% endhighlight %}

Furthermore, there is also a <code>fromF</code> method. I won't go into details about it:

{% highlight java %}
public RouteDefinition fromF(String uri, Object... args)
{% endhighlight %}

The RouteBuilder is closely linked with the <code>org.apache.camel.model.RouteDefinition</code> class. It offers a similar interface concerning the <code>from</code> method, but beyond that also has further support for REST APIs:

{% highlight java %}
public RouteDefinition fromRest(String uri)
{% endhighlight %}

Unfortunately, the API docs are not explaining a lot. Let's assume we wanted to read from a file <code>C:\in\MyFile.txt</code>. Let's be very naive and think that we could actually simply provide the file path to the <code>from</code> (and <code>to</code>) method.


{% highlight java %}
import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

// ...

CamelContext ctx = new DefaultCamelContext();
RouteBuilder route = new RouteBuilder() {
	@Override
	public void configure() throws Exception {
		from("C:\\in\\MyFile.txt")
		.to("C:\\out\\MyFile.txt");
	}
};
ctx.addRoutes(route);
ctx.start();
// Maybe sleep a little here
// Thread.sleep(4000);
ctx.stop();
{% endhighlight %}

What happens when we execute this code? Actually nothing. The code is executed, but nothing is written to the output directory. No exceptions are thrown, not even a warning message is logged. Not quite what we expected, right?

Looking at the API again, we realize that what is needed is actually not a file path but a file URI. Now, being naive again, we look up the [Wikipedia article on file URI schemes](https://en.wikipedia.org/wiki/File_URI_scheme). Obviously, we forgot to provide the required <code>file://</code> URI prefix. So, let's try again (omitting some code for brevity).

{% highlight java %}
public void configure() throws Exception {
	from("file://C:\\in\\MyFile.txt")
	.to("file://C:\\out\\MyFile.txt");
}
{% endhighlight %}

Still does not work. Again, no exception, no warning messages. What's wrong here? Do we need a third slash, i.e. <code>file:///</code>?

{% highlight java %}
public void configure() throws Exception {
	from("file:///C:\\in\\MyFile.txt")
	.to("file:///C:\\out\\MyFile.txt");
}
{% endhighlight %}

Nope, still no success. Maybe double backslashes in file paths are not properly parsed? Next try:

{% highlight java %}
public void configure() throws Exception {
	from("file://C:/in/MyFile.txt")
	.to("file://C:/out/MyFile.txt");
}
{% endhighlight %}

Same result again. This is getting frustrating. All it says in the API documentation of class RouteBuilder:

<blockquote>A <a href="http://camel.apache.org/dsl.html">Java DSL</a> which is used to build DefaultRoute instances in a CamelContext for smart routing.</blockquote>

# Resources
Looking up the website for the [Java DSL](http://camel.apache.org/java-dsl.html) docs does not give a clear hint neither. There exists also a [long manual](http://camel.apache.org/manual/camel-manual-2.16.1.html), but we don't find a lot there neither. And finally, there exists this [documentation on the File2 component](http://people.apache.org/~dkulp/camel/file2.html), which you need to read very carefully to figure out the proper syntax. There's an article on [how to create a file poller and process large files](http://www.javacodegeeks.com/2013/09/exploring-apache-camel-core-file-component.html). There's also [this article](http://kevinboone.net/cameltest.html) which essentially does not say anything beyond what we already know. If you look around a little you may even find the complete book _Apache Camel in Action_ on the internet, nevertheless things stay obscure.

# Working solution
Fast forward. Here's the working solution. As it turns out, Apache Camel does __not__ use traditional file URIs but uses it's own non-standard file URI format. The trick is to specify the filename as a separate parameter added at the end of the directory path.

_file:// + &lt;directory path&gt; + ? + fileName= + &lt;filename&gt; + &amp; + &lt;other optional key=value params&gt;_

For example, if the filename is _C:\in\MyFile.txt_, then the URI would look like one of these (both are valid):

{% highlight java %}
file://C:/in/?fileName=MyFile.txt
file://C:\\in\\?fileName=MyFile.txt
{% endhighlight %}

Let's add a charset parameter to specify the file encoding to be used:

{% highlight java %}
file://C:/in/?fileName=MyFile.txt&charset=utf8
{% endhighlight %}

Here's the full example:

{% highlight java %}
import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

// ...

CamelContext ctx = new DefaultCamelContext();
RouteBuilder route = new RouteBuilder() {
	@Override
	public void configure() throws Exception {
		from("file://C:/in/?fileName=MyFile.txt&charset=utf8")
		.to("file://C:/out/?fileName=MyFile.txt&charset=utf8");
	}
};
ctx.addRoutes(route);
ctx.start();
// Maybe sleep a little here
// Thread.sleep(4000);
ctx.stop();
{% endhighlight %}

# Noop=true
Running this example, we observe something interesting. By default, Apache Camel takes the following sequence of steps:

1. Read the input file _C:/in/MyFile.txt_.
2. Once read, create a new folder _.camel_ inside the input directory and move the input file into this new directory.
3. If the output file does not yet exist, create a new one in the output directory. Otherwise, overwrite the existing one.
4. Write the output file.

If you don't find this behavior useful, then you can adapt it. Let's tell Apache Camel not to create a _.camel_ directory in the input folder but simply leave the input files as they are. This can be achieved with appending the <code>noop=true</code> parameter.

{% highlight java %}
public void configure() throws Exception {
	from("file://C:/in/?fileName=MyFile.txt&charset=utf8&noop=true")
	.to("file://C:/out/?fileName=MyFile.txt&charset=utf8");
}
{% endhighlight %}

There are many more parameters to be used, and they can be looked up in the [documentation of the File2 component](http://people.apache.org/~dkulp/camel/file2.html) mentioned above already.

The good news is, this approach even works for non-text files. Let's assume you need to read from one PDF file and write it to the output directory.

{% highlight java %}
public void configure() throws Exception {
	from("file://C:/in/?fileName=MyFile.pdf&noop=true")
	.to("file://C:/out/?fileName=MyFile.pdf");
}
{% endhighlight %}

It's as easy as this.

# Reading non-text files
This is all good as long as you only intend to process files of the same input and output type. But what if your input file type is different from the target output file type? Neither the core nor the File2 component of Apache Camel provide direct support for such cases. There are different approaches to solve this, but basically all of them come down to file type conversion. Class <code>org.apache.camel.model.RouteDefinition</code> extends class <code>org.apache.camel.model.ProcessorDefinition</code>. ProcessorDefinition in turn offers the following interesting methods:

{% highlight java %}
public Type marshal(DataFormat dataFormat)
public Type marshal(DataFormatDefinition dataFormatDefinition)
public Type marshal(String dataTypeRef)

public DataFormatClause<ProcessorDefinition<Type>> unmarshal()
public Type unmarshal(DataFormat dataFormat)
public Type unmarshal(DataFormatDefinition dataFormatDefinition)
public Type unmarshal(String dataTypeRef)
{% endhighlight %}

In Apache Camel, a DataFormat is an object that can marshal and unmarshal another object from one input type to another. This interface offers only two methods:

{% highlight java %}
void marshal(Exchange exchange, Object graph, OutputStream stream) throws Exception
Object unmarshal(Exchange exchange, InputStream stream) throws Exception
{% endhighlight %}

It's your task to implement these methods properly. Once implemented, you can use your version of DataFormat. Imagine you've written a PdfTextDataFormat that can marshal back and forth between PDF and text files.

{% highlight java %}
public void configure() throws Exception {
	from("file://C:/in/?fileName=MyFile.pdf&noop=true")
	.unmarshal(new PdfTextDataFormat())
	.to("file://C:/out/?fileName=MyFile.txt");
}
{% endhighlight %}

Or the other way round:

{% highlight java %}
public void configure() throws Exception {
	from("file://C:/in/?fileName=MyFile.txt&noop=true")
	.unmarshal(new PdfTextDataFormat())
	.to("file://C:/out/?fileName=MyFile.pdf");
}
{% endhighlight %}

To implement your PdfTextDataFormat's unmarshal method you must:

1. read the raw file content from the input stream provided,
2. convert the raw data to a text string,
3. set the text string as the body of the exchange's out message.

Your code should look something like this:

{% highlight java %}
import org.apache.camel.spi.DataFormat;
import org.apache.commons.io.IOUtils;

public PdfTextDataFormat implements DataFormat {

	public void marshal(Exchange exchange, Object graph, OutputStream stream) { ... }

	public Object unmarshal(Exchange exchange, InputStream stream) throws Exception {
		byte[] bytes = IOUtils.toByteArray(stream);

		// Use a tool like PDFBox to create text from your bytes.
		String text = ...;
		Message out = exchange.getOut();
		out.setBody(text);

		// Don't close input stream here
	}
}
{% endhighlight %}

The marshalling method would probably look something like this:

{% highlight java %}
import org.apache.camel.spi.DataFormat;
import org.apache.commons.io.IOUtils;

public PdfTextDataFormat implements DataFormat {

	public void marshal(Exchange exchange, Object graph, OutputStream stream) {
		// Don't do this: String s = (String) o;
		// Instead, use Camel type converters like this:
		String s = exchange.getContext().getTypeConverter().mandatoryConvertTo(String.class, graph);
		
		// Create a PDF document from the string and convert it into a byte array
		byte[] bytes = ...;

		IOUtils.write(bytes, stream);

		// Don't close output stream here
	}

	public Object unmarshal(Exchange exchange, InputStream stream) throws Exception { ... }
}
{% endhighlight %}

In case you only want to do (un-) marshalling in one direction but not in both, it may be a better idea to write a converter processor implementing the <code>org.apache.camel.Processor</code> interface.

Fortunately, you don't really need to build your own PDF-to-text data format. Instead, you may want to use the [camel-tika component](https://github.com/wheijke/camel-tika). This component is able to unmarshal text from various binary formats (including MS Office documents) to plain text (but not marshalling them in the opposite direction):

{% highlight xml %}
<dependendy>
	<groupId>org.apache.camel</groupId>
	<artifactId>camel-tika</artifactId>
	<!-- <version>0.2</version> -->
</dependenc>
{% endhighlight %}

You may have to update camel-tika's pom.xml though, as it seems to not have been updated in a while.

Here's another blog post on [how to do marshalling](http://blogs.sourceallies.com/2013/02/getting-started-with-camel-marshalling/).
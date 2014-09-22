---
layout: post
title: Validation in JSONForm
comments: true
tags: [programming, jsonform]
---
Recently, I had created a user form with [JSONForm](https://github.com/joshfire/jsonform). However, the form was embedded in another site with its own <code>save</code> button. JSONForm usually adds its own <code>submit</code> button to the site, but [you easily can remove it](https://github.com/joshfire/jsonform/wiki#fields-submit). One of the problems left was how to trigger validation for the form manually once the site's <code>save</code> button was clicked. Be aware that this implies triggering validation from outside the HTML <code>form</code> element. Whereas JSONForm's <code>submit</code> button resides inside the <code>form</code> element as an <code>&lt;input type="button"&gt;</code>, my site's <code>save</code> button does not. Unfortunately, the description in the JSONForm's Wiki pages was not really understandable to me. It took me a long time but I finally succeeded to trigger JSONForm validation manually. Here's what I did.<!--more-->

JSONForm's validation relies on [its own adapted version of JSV.js](https://github.com/joshfire/jsonform/blob/master/deps/opt/jsv.js). You have to make sure that this library is accessible to your JSONForm code. In the simplest case you can include the Javascript in your HTML site before the jsonform library. (I renamed the file to <code>jsonform-jsv.js</code> to make clear that this is a modified version of JSV.js.)
{% highlight html %}
<html>
<head></head>
<body>
    <script type="text/javascript" src="deps/jquery.min.js"></script>
    <script type="text/javascript" src="deps/underscore.js"></script>
    <script type="text/javascript" src="deps/opt/jsonform-jsv.js"></script>
    <script type="text/javascript" src="lib/jsonform.js"></script>
</body>
{% endhighlight %}
In my case though all Javascript files were pre-compiled into a single, big, compressed file with [Browserify](http://browserify.org/) and only this file was referenced from my HTML file. All Javascript libraries followed the [Requirejs](http://requirejs.org/) standard. Hence, the - theoretically - correct way to add another library such as JSV.js would be to use a <code>require('JSV');</code> statement. In practice however this would not work, because doing so would actually imply loading the official version of JSV.js instead of JSONForm's own adapted version. I finally settled with the solution to add the JSONForm's version inside HTML together with the pre-compiled Javascript file:
{% highlight js %}
<html>
<head></head>
<body>
    <script type="text/javascript" src="jsonform-jsv.js"></script>
    <script type="text/javascript" src="my-precompiled-js-lib.js"></script>
</body>
{% endhighlight %}
Maybe not the most elegant solution, but it worked. It is important to understand that if you do not provide a JSV library, then clicking JSONForm's <code>submit</code> button will actually trigger the HTML5 compliant browser-internal form validation. This is not really what you want to happen, as there are some important differences between JSONForm's own validation procedure and the one provided internally by most modern browsers. For instance, JSONForm renders input elements with <code>type="number"</code> actually as text inputs but validates them as number inputs, whereas the HTML5 compliant browser-internal validation validates them as text input.

So, I still had to trigger the validation manually. This involved several steps. The problem is that there exists code for validation inside the JSONForm Javascript library, but unfortunately it is tightly bound to the <code>submit</code> button click. I did not want to change any code inside the JSONForm library and try to expose the validation function to the outside world. There are however two other exposed functions that I could rely upon: <code>myFormEl.jsonFormValue()</code> and <code>myFormEl.jsonFormErrors(errors, options)</code>. The first method returns all entered form values, the second one highlights invalid form input elements.

Thus, in my site's <code>save</code> function, added the following code:
{% highlight js %}
var formEl = $('#myForm');
var env = JSONFormValidator.createEnvironment("json-schema-draft-03");
var schema = { 'properties': myJsonForm.schema };
var report = env.validate(formEl.jsonFormValue(), schema);
var options = {};
formEl.jsonFormErrors(report.errors, options);
{% endhighlight %}

There's quite a lot contained in this code snippet, so let's break it down.

1. <code>var formEl = $('#myForm');</code>  
This line uses JQuery to get the HTML form element with the id of <code>#myForm</code>. You could achieve this without JQuery (e.g. <code>document.getElementById('myForm')</code>, but since JSONForm requires JQuery anyway, why not use it?

2. <code>var env = JSONFormValidator.createEnvironment("json-schema-draft-03");</code>  
Here we create a new JSV validator environment. This assumes that the variable <code>JSONFormValidator</code> is globally accessible. Alternatively, <code>JSV.createEnvironment("json-schema-draft-03")</code> would achieve the same.

3. <code>var schema = { 'properties': myJsonForm.schema };</code>  
This one took me long to figure out. <code>myJsonForm.schema</code> refers to the JSONForm's <code>schema</code> property you had to define for creating the HTML form. Be aware how I wrapped the schema inside another object as a value of a property called <code>properties</code>. This is important! If you simply use <code>myJsonForm.schema</code> directly, the validation will never indicate any errors.

4. <code>var report = env.validate(formEl.jsonFormValue(), schema);</code>  
This code actually triggers the validation. We retrieve the user input with <code>formEl.jsonFormValue()</code> and use it together with the schema as an input to the validation function. Report will be an object containing an <code>errors</code> property, this is what we will need for further processing. If there are no errors, then <code>report.errors.length</code> will be 0. If it has length > 0 then there are errors. I used this information to abort the saving procedure.

5. <code>var options = {};  
formEl.jsonFormErrors(report.errors, options);</code>  
<code>options</code> is simply an empty object. In the current JSONForm version, it does not serve any purpose, but it is still needed as a function parameter. Calling <code>formEl.jsonFormErrors(...)</code> will highlight invalid input elements on the page and also show a short error message to the user.

Some remarks. First, JSONForm's logic how to render certain specified input types to input elements is not always intuitive. For example, the schema specification <code>"foo": { "type": "number" }</code> results in an input element of type text: <code>&lt;input type="text"&gt;</code>. [JSONForm on the default mapping](https://github.com/joshfire/jsonform/wiki#default-mapping):
<blockquote><ul>
<li>A <code>number</code> property generates a text input, i.e. an &lt;input type="text"&gt; element.</li>
<li>An <code>integer</code> property generates a text input as well, i.e. an &lt;input type="text"&gt; element.</li></ul></blockquote>
What is even stranger is that specifying <code>type="number"</code> in the form section atually renders <code>&lt;input type="number"&gt;</code> elements.

Second, having an input fields set to <code>readonly</code> resulted in validation errors if I had specified this inside the schema section. It worked though if specified inside the form section.

<table>
  <tr>
    <th>Did not work</th>
    <th>Worked</th>
  </tr>
  <tr>
    <td>
{% highlight js %}
{
  "schema": {
    "foo": {
      "type": "string",
      "readonly": "readonly"
    }
  },
  "form": [{
    "foo"
  }],
  "onSubmitValid": function(values) {}
}
{% endhighlight %}
    </td>
    <td>
{% highlight js %}
{
  "schema": {
    "foo": {
      "type": "string
    }
  },
  "form": [{
    {
      "key": "foo",
      "readonly": "readonly"
    }
  }],
  "onSubmitValid": function(values) {}
}
{% endhighlight %}
    </td>
  </tr>
</table>

Third, when I was using [JSONForm's arrays of objects](https://github.com/joshfire/jsonform/wiki#fields-arrays), setting an object property to <code>required</code> in the schema section resulted in validation errors. If however specified in the form section, this did not seem to have any effect whatsoever, that is apparently no check was performed whether the required input fields were filled out or not.
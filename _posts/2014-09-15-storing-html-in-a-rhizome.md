---
layout: post
title: Storing HTML in a Rhizome
comments: true
---

This post demonstrates how it is possible to use rhizomes to store simple HTML.

Consider the following HTML.
{% highlight html %}
<html>
<head></head>
<body></body>
</html>
{% endhighlight %}
How could we store this in a rhizome? First of all, it would make sense to treat every HTML tag as an atomic symbol. There are three such symbols in the sample: _html_, _head_ and _body_.<!--more--> Let us assume that, unless qualified otherwise, the direction of a relation indicates a parent-child relationship. Thus, _(x, y)_ must be read as "x _is parent of_ y". The _html_ tag has two children and they are ordered. _html_ is both parent of _head_ and of _body_, but _head_ is the first child and _body_ is the second. How could we express this fact? At this time, it is useful to introduce _qualifiers_.


__Definition:__
<div class="message">A _qualifier_ _q_ is a relation which, when paired with another relation _r<sub>i</sub>_, indicates how that relation _r<sub>i</sub>_ should be processed.</div>
We will use square brackets <code>[]</code> to denote a relation as a qualifier.

In the next table terminal relations were assigned to each HTML tag. Additionally, two qualifiers were introduced, one for child relations and a second one for sibling relations.

<table>
  <tr>
    <th>HTML Tag</th>
    <th>Terminal Relation</th>
  </tr>
  <tr>
    <td>[child]</td>
    <td>(0, 0)</td>
  </tr>
  <tr>
    <td>[sibling]</td>
    <td>(1, 1)</td>
  </tr>
  <tr>
    <td>html</td>
    <td>(2, 2)</td>
  </tr>
  <tr>
    <td>head</td>
    <td>(3, 3)</td>
  </tr>
  <tr>
    <td>body</td>
    <td>(4, 4)</td>
  </tr>
</table>

It is now possible to express the HTML above as two different, subsequent relations:

1. <code>((html, [child]), head) = (((2, 2), (0, 0)), (3, 3))</code>
2. <code>((html, [child]), body) = (((2, 2), (0, 0)), (4, 4))</code>

Or expressed tree-like:

<table>
  <tr>
    <td>
{% highlight js %}
(
  (
    html,
    [child]
  ),
  head
)
(
  (
    html,
    [child]
  ),
  body
)
{% endhighlight %}
    </td>
    <td>
{% highlight js %}
(
  (
    (2, 2),
    (0, 0)
  ),
  (3, 3)
)
(
  (
    (2, 2),
    (0, 0)
  ),
  (4, 4)
)
{% endhighlight %}
    </td>
  </tr>
</table>

Note that we did not add a comma <code>,</code> between the head and the body relation. Adding a [sibling] qualifier, we pair the two relations and end up with:

1. <code>((((html, [child]), head), [sibling]), ((html, [child]), body)) = (((((2, 2), (0, 0)), (3, 3)), (1, 1)), (((2, 2), (0, 0)), (4, 4)))</code>

Or expressed as a tree:

<table>
  <tr>
    <td>
{% highlight js %}
(
  (
    (
      (
        html,
        [child]
      ),
      head
    ),
    [sibling]
  ),
  (
    (
      html,
      [child]
    ),
    body
  )
)
{% endhighlight %}
    </td>
    <td>
{% highlight js %}
(
  (
    (
      (
        (2, 2),
        (0, 0)
      ),
      (3, 3)
    ),
    (1, 1)
  ),
  (
    (
      (2, 2),
      (0, 0)
    ),
    (4, 4)
  )
)
{% endhighlight %}
    </td>
  </tr>
</table>


{% highlight html %}
<html>
<head>
  <title>Hello World!</title>
</head>
<body>
  <h1>My First Heading</h1>
</body>
</html>
{% endhighlight %}

Using only the simple rules above, we can now go on adding tags. Each tag is either a child or a sibling in relation to another one.

Imagine that two clients store the same atomic symbol table and use the same encoding algorithm. It is now possible to send a single (possibly very long) integer number over a network, and the receiver can fully re-compute the complete HTML tree. Of course, a possibility for actually storing content is still missing. For this purpose, we could introduce a further qualifier [value]. Once the atomic symbol table is complete concerning valid HTML tags, we start adding one numbered variable per plain, textual content. A relation <code>(s<sub>i</sub>, [value])</code> would indicate a textual content stored in variable _s<sub>i</sub>_. Of course it would be necessary to also submit the plain textual content of every variable over the network. The same procedure would be applicable to tag attributes. First, introduce a new qualifier [attribute]. Then, add all valid HTML attributes (such as id, name, href etc.) to the atomic symbol table. (Be aware that this does not prohibit us to create meaningless combinations of tags and attributes such as <code>&lt;table href=""&gt;</code>.) The textual content variables are then stored after the tag attributes in the atomic symbol table.
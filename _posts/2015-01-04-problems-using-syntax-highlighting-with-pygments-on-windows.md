---
layout: post
title: Problems Using Syntax Highlighting with Pygments on Windows
comments: true
tags: [jekyll, pygments]
---
I wanted to turn on code syntax highlighting using rouge for my blog by adding the following line to my _&#95;config.yml_ file.
{% highlight yaml %}
highlighter:   rouge
{% endhighlight %}
Whereas this worked perfectly on my local Windows machine, I ran into problems with Jekyll on GitHub.<!--more--> After searching for some time, I found [this article](http://www.codeproject.com/Articles/809846/Blogging-on-GitHub) from August 2014 where it stated:
<blockquote>
__Update: As of August 1, commiting a__ &#95;config.yml __that uses__ rouge __now causes "Page build failure" on GitHub with a misleading error message like "The file__ &#95;posts/2014-08-01-blah.md __contains syntax errors."__ Before you commit & push, you must set highlighter: pygments in &#95;config.yml, even if you don't care to install pygments locally.
</blockquote>
It's now January 2015, and this was exactly the error message I received. So I figured there was/is a problem with rouge running on GitHub's Jekyll and decided to use pygments instead. My _&#95;config.yml_ file:
{% highlight yaml %}
highlighter:   pygments
{% endhighlight %}
Whereas the change went through smoothly remotely on GitHub's Jekyll, I ran into problems locally. Whenever I tried to <code>jekyll build</code> my site, I received this error message:
{% highlight shell-session %}
Liquid Exception: No such file or directory - python2 C:/Ruby200-x64/lib/ruby/gems/2.0.0/gems/pygments.rb-0.6.0/lib/pygments/mentos.py in _posts/2000-01-01-my-oldest-post.md
{% endhighlight %}
I did not know what this meant and it took me some hours to figure it out. The problem is that for any reason pygments is looking for a file _python2.exe_ and cannot find it. I am using an Enthought Canopy python distribution, and what I _do_ have is a file <code>C:\Enthought-Canopy\User\python.exe</code> as well as <code>C:\Enthought-Canopy\User\Scripts\python.exe</code>. First, I had to make sure that both directories containing these files were in my <code>System PATH</code> variable. Do not confuse with the <code>User PATH</code> variable. (Also don't forget to close and reopen the Windows command prompt afterwards, otherwise your changes are not effective.) Then [I had to make a simple copy of python.exe and name it python2.exe](http://stackoverflow.com/questions/17364028/jekyll-on-windows-pygments-not-working). Alternatively, I could have created a hard link inside the same directory where python.exe exists and name it python2.exe.
{% highlight shell-session %}
cd C:\Enthought-Canopy\User
mklink /H python2.exe python.exe

cd C:\Enthought-Canopy\User\Scripts
mklink /H python2.exe python.exe
{% endhighlight %}
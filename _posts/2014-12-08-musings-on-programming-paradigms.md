---
layout: post
title: Musings on Programming Paradigms
comments: true
tags: [programming, object-oriented programming, clojure]
---
Yesterday I had to fill out a questionnaire on my programming skills as part of a job application procedure. I was asked to name some GoF and JEE patterns I am familiar with, so I pondered on what I had learned for a little while. One thing that somehow struck me as odd was the silent underlying assumption that a "good programmer" nowadays is actually supposed to be familiar with these patterns. Singleton, MVC, Factory, Adapter, Fly-Weight, DAO and so on. After all, programming is a lot about robust design and improving code quality, isn't it? And sure it is. There is nothing more frustrating than trying to untangle an old piece of software code horribly designed and possibly poorly documented. Using OOP patterns appropriately simply makes the world a better place. So, where is the problem?<!--more-->

The problem is: On a cloudy day in fall roughly two years ago I decided to learn the Clojure programming language. I am not an expert here, I hurry to add. But learning Clojure is like a [Satori in Zen](http://en.wikipedia.org/wiki/Satori). Once you've had one, the world is never going to be exactly the same. Clojure is different. Not only is it a functional programming language with a weird syntax (there are others like Scala or Python) but it is also [homoiconic](http://en.wikipedia.org/wiki/Homoiconicity). And there are macros. It is the combination that makes the difference.

If you haven't yet stumbled upon them, I really recommend reading the following two articles on a quiet Sunday afternoon.

* [Paul Graham - Revenge of the Nerds](http://www.paulgraham.com/icad.html)
* [Steve Yegge - Execution in the Kingdom of Nouns](http://www.eecis.udel.edu/~decker/courses/280f07/paper/KingJava.pdf)

Of course, both pieces are a little polemic. Java is not bad. In fact, it is much better than many other programming languages existing before it. (Due to its simplicity and platform independency I personally prefer it over C++, for example. But that's a matter of taste.) Yet, once you have an impression on how much less boiler plate code is actually needed in a functional language like Scala or Clojure, you start feeling envious. In Clojure you actually think about your function for a considerable amount of time, then you sit down and write a few lines of code so powerful that it would need up to 10x the amount of code in Java to achieve the same. You don't believe me? [Read this article](http://redmonk.com/dberkholz/2013/03/25/programming-languages-ranked-by-expressiveness/).

In his article, Paul Graham concludes:
<blockquote>When I see patterns in my programs, I consider it a sign of trouble. The shape of a program should reflect only the problem it needs to solve. Any other regularity in the code is a sign, to me at least, that I'm using abstractions that aren't powerful enough-- often that I'm generating by hand the expansions of some macro that I need to write.</blockquote>

Here we are at the core of the problem. To me, using OOP patterns is often a sign of a programming language's expressive weakness rather than an indicator of beautiful design. Many OOP patterns are simply obsolete when using a functional language, you don't need them because the problem addressed by the design pattern typically does not or cannot arise. Of course this does not mean that I don't use OOP patterns a lot. I certainly do, and I recommend every programmer doing so.  
I am also aware that Java 8 introduces its own version of [lambda expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html). I remain a little skeptical. In a language like Clojure, the majority of my code is functional and only in certain places I deviate from the paradigm. In an OOP language like Java, the majority of code is object-oriented, even if in fact it makes little sense (as in <code>new Importer().import()</code> or or in <code>new MyFunction().execute()</code>). To me, having lambda expressions in Java is a step in the right direction and I will happily use them as soon as I can, but having lambda expressions still does not make Java a functional programming language. And [I'm not the only person thinking this way](http://www.beyondjava.net/blog/java-8-functional-programming-language/).

If you would like to have a glimpse of what is possible with functional programming languages in combination with NoSQL databases watch this video:

<center><iframe src="//player.vimeo.com/video/15955920" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe></center>
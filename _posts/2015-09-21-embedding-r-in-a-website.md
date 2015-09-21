---
layout: post
title: Embedding R In A Website
comments: true
tags: [R]
---
I wanted to know whether/how it is possible to embed R in a website. Looking around the internet I found a few interesting initiatives, each one dedicated to a slightly different purpose: RStudio, Shiny, Jupyter Notebook, RApache, OpenCPU and RAppArmor.<span class="more"></span>

[RStudio](https://www.rstudio.com/) is probably very well known among R programmers. According to its website, RStudio is an integrated development environment. One of the best features of RStudio in my eyes is the ability to create both markdown and HTML pages, that contain executable R code. If you are familiar with [IPython notebook] then this is nothing new for you. In this way you can easily create protocols of your work, share them with colleagues and publish them in a webpage.  
RStudio comes both as a standalone desktop edition and a server edition for projects of bigger scale. Both desktop and server are available as a free open source product and as a commercially licensed product.

[Shiny](http://shiny.rstudio.com/) I found pretty exciting. It is essentially a web application framework that lets you build interactive web apps quickly and easily without having to have extensive web programming skills. You can either run your Shiny websites in your own web server or use the cloud hosting service provided by the vendor.  
Shiny is developed by the same company that sells RStudio. The open source version of Shiny [can be obtained through this GitHub page](https://github.com/rstudio/shiny). I believe there's also a cloud where you can upload your R web apps.

[Jupyter Notebook](http://jupyter.org/) looks very impressive to me. Being essentially an extension of the already mentioned IPython Notebook (and thus requiring Python to run) you can write research reports easily, mix comments with executable code etc. Besides Python and R many more languages are supported like Scala, Julia or Haskell.

[rApache](http://rapache.net/) enables you to run R inside your Apache web server in a script-like fashion. rApache distributes the Apache module _mod\_R_ for this purpose. Therefore you have a neat integration of R with the probably most relevant open source web server. It's not entirely clear to me how secure this is, therefore you should probably take the usual precautions before opening a webpage to the whole world that lets the users execute such code on your server.

[OpenCPU](https://www.opencpu.org/) is both an API plus server. On the one hand, it defines a REST-API that enables you to call R code on any server implementing this standard. As it is RESTful, you can easily create HTTP requests executing your R commands in an intuitive way. On the other hand, OpenCPU is also a server. The server can be set up on top of for example rApache or RStudio server. It handles object serialization, security, resource control and more. [An overview on OpenCPU was published in this paper on arxiv.org](http://arxiv.org/abs/1406.4806), there is also [a slide show giving a summary](http://jeroenooms.github.io/opencpu-slides/#1).

Finally, [RAppArmor](https://cran.r-project.org/web/packages/RAppArmor/index.html) is a [CRAN package] that aims at securing your R execution engine. Whereas the concept of sandboxing has existed for example for the Java Virtual Machine already from the beginning, the same is not the case for R. As soon as you run R on the server and open it up for the rest of the world, you of course would like to have a certain level of control over resources and execution rights. RAppArmor was built for this purpose.

Of course there's even more going on with projects like [rpy2](http://rpy.sourceforge.net/rpy2.html) or [Rocker](http://dirk.eddelbuettel.com/blog/2014/10/23/), but most of them do not explicitly target integration with the web.
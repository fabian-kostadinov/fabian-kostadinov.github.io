---
layout: post
title: Installation of Jekyll-Auth
comments: true
tags: [jekyll-auth]
---
These are (hopefully) complete installation instructions for Jekyll-Auth. To understand how Jekyll-Auth works, you need a conceptual understanding on how [Rack-Jekyll](https://github.com/adaoraul/rack-jekyll/) and [Rack](https://github.com/rack/rack) work. The next figure shows the conceptual workflow of Jekyll-Auth in combination with a repository on GitHub.com. This is how Jekyll-Auth works.<span class="more"></span>

![Jekyll-Auth Workflow](/public/img/2014-11-13-installation-of-jekyll-auth.png "Jekyll-Auth Workflow")

* On GitHub.com, there exists an organization _foo-organization_ containing a team _foo-team_ and a repository _foo-repository_. _foo-repository_ is private, therefore only members of _foo-team_ can view or modify its contents.
* Although the repository contains a complete Jekyll-enabled website, GitHub pages is not used to host any of this content. Instead, all users only work on the <code>master</code> branch (or any other branch except a GitHub pages <code>gh-pages</code> branch). 
* The website's content is not accessible publicly. Instead, on Heroku an app is running asking for user authentication when someone tries to access the website. This Heroku app is a Jekyll server enhanced with Jekyll-Auth functionality.
* In the example, there are currently two users pushing and pulling to _foo-repository_ on GitHub.com: Anna and Bob. Both users are members of _foo-organization_ and of _foo-team_.
* Bob is an ordinary GitHub user. He works on a local clone of _foo-repository_. Whenever he has changes, he simply pushes them to the remote GitHub repository.
* Anna is also a GitHub user, hence she has her own local clone of _foo-repository_ like Bob. However, unlike Bob, she is also responsible for hosting the website's content on Heroku so that it is really accessible as a fully functional website (and not only as files) to all members of _foo-team_.
* The local version of the GitHub repository will therefore contain both the common website files and the special files provided by Jekyll-Auth. Bob does not care much about these additional files. But Anna has two responsibilities. Like Bob she can work on the website's content. But at certain predefined points in time, she can also update the hosted website running on Jekyll as a Heroku app.
* Heroku is _not_ notified of changes going on inside _foo-repository_ on GitHub. Anna needs to manually perform a <code>git push heroku master</code> every time she wants to update the hosted website on Heroku.
* The hosted website is accessible under the URL <code>https://my-foo-heroku.herokuapp.com</code>. Whenever a user wants to access this site, Heroku redirects him to a GitHub authorization page, where he must provide his GitHub username and password. Heroku also sends Client ID, Client Secret as well as organization ID and/or team ID (e.g. "@foo-organization/foo-team") to GitHub. GitHub now tries to authenticate the user, and if he is a member of _foo-organization_ and _foo-team_ and is allowed to access _foo-repository_, then he is granted access to the website's content. In the example, Charles is no organization/team member and is denied access to the website.
* Access rights to files and folders are specified inside the _foo-repository_'s _&#95;config.yml_ file.

#Installation instructions

This is a step-by-step installation instruction.

----

_Prerequisites:_ Before you begin, you will need a repository that contains your website's content, that is all the HTML and CSS files, JavaScript code, your images etc. Make sure your local clone is up to date with your remote GitHub repository. I will call this your _website repository_.

----

_Step 1:_ Make sure you have a [Heroku account](http://www.heroku.com). A free one will be sufficient for most needs.

----

_Step 2:_ Make sure you have [Heroku Toolbelt](https://toolbelt.heroku.com/) installed. You will probably need to use your Heroku login information. Further down, you will run <code>bundle install</code>. If you haven't Heroku Toolbelt installed but instead use Heroku gem, the installation will not work properly and you will receive this warning message:
{% highlight console linenos %}
Your bundle is complete!  
Use `bundle show [gemname]` to see where a bundled gem is installed.  
Post-install message from heroku:  
 !    The `heroku` gem has been deprecated and replaced with the Heroku Toolbelt.  
 !    Download and install from: https://toolbelt.heroku.com  
 !    For API access, see: https://github.com/heroku/heroku.rb
{% endhighlight %}
If this shows up, you need to first uninstall Heroku gem: <code>gem uninstall heroku</code>. Heroku gem is deprecated and it will interfere with your Heroku Toolbelt installation, so make sure you actually uninstalled it.

----

_Step 3:_ If you've just installed Heroku Toolbelt, you will probably have to recreate SSH keys, otherwise your local Heroku Toolbelt will not be able to push files to the Heroku server. There is this [article at heroku.com on the use of SSH keys](https://devcenter.heroku.com/articles/keys) if you want to know more about this. Create a key <code>ssh-keygen -t rsa</code>, then add the key to Heroku <code>heroku keys:add</code>. Make sure that you __do not mistakenly publish your private RSA key file__ together with the rest of the website!

----

_Step 4:_ Use Heroku Toolbelt to [create a new Heroku app](https://devcenter.heroku.com/articles/creating-apps): <code>heroku create my-new-cool-heroku-app</code>. Your website will be available at <code>https://my-new-cool-heroku-app.herokuapp.com</code>, and it will have a Heroku git account to push to called <code>git@heroku.com:my-new-cool-heroku-app.git</code>. You have to understand thus that, when inside the repository and once you have added files and committed them with <code>git commit -m "Blah"</code>, you can either push to the Heroku git account using the command <code>git push heroku master</code> or to the GitHub remote repository by using the command <code>git push</code>.

Important: Creating a Heroku app in this way will automatically deploy it to servers in the USA. [As is explained in this tutorial on devcenter.herokuapp.com](https://devcenter.heroku.com/articles/app-migration), in case you want the app to run on servers located in Europe, you must first create a new dummy app, create a fork with the corrected region and then delete the dummy app.
{% highlight shell-session %}
# First create a dummy app running on US servers
heroku create my-dummy-app-in-us

# The next line will create fork of the first app, but running on Heroku servers in Europe
heroku fork -a my-dummy-app-in-us my-new-cool-heroku-app --region eu

# Don't forget to delete the dummy app afterwards
heroku apps:destroy my-dummy-app-in-us

 !    WARNING: Potentially Destructive Action
 !    This command will destroy my-dummy-app-in-us (including all add-ons).
 !    To proceed, type "my-dummy-app-in-us" or re-run this command with --confirm my-dummy-app-in-us
> my-dummy-app-in-us
Destroying my-dummy-app-in-us (including all add-ons)... done
{% endhighlight %}

----

_Step 5:_ The Heroku app will access the GitHub account to perform an authorization check for every user. If the user is registered with the corresponding GitHub account, she will also be allowed to access the Heroku app. Hence, the Heroku app must be registered with GitHub. Upon registration, you will receive a OAuth2 Client ID and Client Secret which will be needed at a later step.

Login to your organization's GitHub account, i.e. something like <code>https://github.com/organizations/foo-organization/settings/applications</code>. Click on _Register new application_. Enter the following information:

* _Application name:_ Any meaningful name for this application.
* _Homepage URL:_ The link to your heroku app received in step 3, e.g. <code>https://my-new-cool-heroku-app.herokuapp.com</code>.
* _Application description:_ A textual description.
* _Authorization callback URL:_ Same as homepage url + <code>/auth/github/callback</code> appended, e.g. <code>https://my-new-cool-heroku-app.herokuapp.com/auth/github/callback</code>

Attention: The correct Heroku URL necessarily starts with <code>https://...</code> and not with <code>http://...</code>.

You will be given a Client ID and a Client Secret, that is a shorter and a longer string of numbers and letters. We will need them later on, so you better write them down. In case you want to know what they are useful for, here's a short excerpt from [OAuth's API description](https://developer.github.com/v3/oauth/):
<blockquote>OAuth2 is a protocol that lets external apps request authorization to private details in a user's GitHub account without getting their password. This is preferred over Basic Authentication because tokens can be limited to specific types of data, and can be revoked by users at any time.

All developers need to register their application before getting started. A registered OAuth application is assigned a unique Client ID and Client Secret. The Client Secret should not be shared.</blockquote>

----

_Step 6:_ Make sure you have Ruby installed. Jekyll-Auth depends on Ruby (and other stuff).

----

_Step 7:_ Make sure you have [Ruby's bundler](http://bundler.io/) installed. (To check if you have, simply call <code>bundle --version</code> in your shell.)


----

_Step 8:_ Download the content of [the original Jekyll-Auth repository from GitHub](https://github.com/benbalter/jekyll-auth). Whether you create a local clone or download it as a zip-file does not matter. copy all the files from the downloaded Jekyll-Auth repo (except the hidden _.git_ directory and everything contained in it) into your local website repository. The local website repository now contains both your website content and the content obtained from Jekyll-Auth. (If you created a local clone of the Jekyll-Auth repository, you can delete it now. It will no longer be used.)

----

_Step 9:_ Navigate to your local website repository. There should be a <code>Gemfile</code> in your repository's directory. Change this file so that it looks like this:
{% highlight Ruby %}
source "https://rubygems.org"

gem 'jekyll-auth'
{% endhighlight %}

If you receive an error message stating 'certificate verify failed' this refers to using _https_ instead of _http_ in your Gemfile. In case you don't care about a secured connection, change the Gemfile like this:
{% highlight Ruby %}
source "http://rubygems.org"

gem 'jekyll-auth'
{% endhighlight %}

Another problem suddenly occurred connected to this step, and it took me quite some time to figure out what the problem was. At one point pushing my local repository to Heroku with <code>git push heroku master</code> suddenly failed, the error message said something about a method <code>ssl?</code> being not found. After searching for a significant amount of time I found [an article on how to deploy Ruby applications to Heroku from a Windows environment](https://devcenter.heroku.com/articles/bundler-windows-gemfile). The problem is that if you are working on a Windows machine (as I do) then you _must_ specify a version in your _Gemfile_. Your entry should thus look similar to this:
{% highlight Ruby %}
source "https://rubygems.org"

gem 'jekyll-auth', '0.6.1'
{% endhighlight %}
Heroku is running a Linux environment, and some required libraries differ from the ones used in Windows. If you don't specify a version, then <code>git push heroku master</code> will call <code>rake assets:precompile</code> which again will call the <code>bundle exec jekyll-auth build</code> command. The Gemfile specified locally in your Windows environment will create a _Gemfile.lock_ file, which contains a list of the exact versions used locally. But this file will not be submitted from your local Windows computer to Heroku's Linux remote environment, only the Gemfile will be used. By default, the Gemfile simply downloads the most current version of Jekyll-Auth from somewhere, which can be a different version than the one you used locally. This behavior can lead to all sorts of strange compilation problems. Hence, you really should specify a Jekyll-Auth version. You can actually look on your local computer which versions of Jekyll-Auth are installed. On my machine I can find all different versions under <code>C:\Ruby200-x64\lib\ruby\gems\2.0.0\gems</code>.

----

_Step 10:_ Then run <code>bundle install</code>. You might see a warning that <code>DL is deprecated, please use Fiddle</code>, which you can safely ignore.

----

_Step 11a:_ Still inside your local clone's directory, you can now run <code>jekyll-auth new</code> to create a new Heroku app. Follow all these steps.

----

_Step 11b:_
{% highlight console %}
...
Would you like to set up Heroku now? (Y/n)
{% endhighlight %}
Type <code>Y</code> and hit <code>Enter</code>.

----

_Step 11c:_
{% highlight console %}
If you already created an app, enter it's name
otherwise, hit enter, and we'll get you set up with one.
Heroku App name?
{% endhighlight %}
We have already created a Heroku app in step 3 with the name _my-new-cool-heroku-app_. Type <code>my-new-cool-heroku-app</code> and hit <code>Enter</code>.

----

_Step 11d:_
{% highlight console %}
...
Git remote heroku added
Awesome. Let's teach Heroku about our GitHub app.
What's your GitHub Client ID?
{% endhighlight %}
Here we need to enter the GitHub OAuth Client ID. Copy and paste the Client ID you received in step 5.

----

_Step 11e:_
{% highlight console %}
...
What's your GitHub Client Secret?
{% endhighlight %}
Then enter the GitHub Client Secret. Copy and paste the Client Secret you received in step 5.

---

_Step 11f:_
{% highlight console %}
...
What's your GitHub Team ID?
{% endhighlight %}
Enter the GitHub Team ID. The team id is an integer number consisting of roughly six or seven digits, like <code>1234567</code>. Using the team's name will _not_ work! I have created [an extra post how to find your team id]({% post_url 2015-01-16-how-to-find-a-github-team-id %}). Be aware that you _cannot_ use a private (paid or unpaid) account's username, it _must_ be a team created with an organizational account.

----

_Step 12:_
We are not yet ready to push our local clone of Jekyll-Auth to the remote Heroku server. We still need to add the Gemfile.lock to the repository:
{% highlight console %}
git add -f Gemfile.lock
git commit -m "Added Gemfile.lock"
{% endhighlight %}
Be aware that we use the <code>-f</code> parameter to enforce adding this file. If you do not provide the parameter git might refuse to add the file to the repository because it is actually ignored in <code>.gitignore</code>. In this case you might end up with the following error message:
{% highlight shell-session %}
git push heroku master

Counting objects: 46, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (37/37), done.
Writing objects: 100% (46/46), 10.14 KiB, done.
Total 46 (delta 6), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Ruby app detected
remote: -----> Compiling Ruby/NoLockfile
remote:  !
remote:  !     Gemfile.lock required. Please check it in.
remote:  !
remote:
remote:  !     Push rejected, failed to compile Ruby app
remote:
remote: Verifying deploy...
remote:
remote: !       Push rejected to edb-website.
remote:
To https://git.heroku.com/my-new-cool-herokuapp.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://git.heroku.com/my-new-cool-herokuapp.git'
{% endhighlight %}

----

_Step 13:_
In the _&#95;config.yml_ file you can specify which directories and files are accessible without authentication and which are not. By default, all access requires authorization except for the _drafts_ directory. You can actually use regular expressions to specify the files and directories. The following denies access to all parts of the site except for the _drafts_ directory.
{% highlight yaml %}
jekyll_auth:
  whitelist:
    - drafts?
{% endhighlight %}
Using regexes, you can also reverse the logic, allowing access to everything except _drafts_:
{% highlight yaml %}
jekyll_auth:
  whitelist:
    - "^((?!draft).)*$"
{% endhighlight %}

----

_Step 14:_
Now we are finally ready to push everything to the remote Heroku server: <code>git push heroku master</code>.

----

_Step 15:_
Open a browser and navigate to the Heroku URL <code>https://my-new-cool-herokuapp.herokuapp.com</code>. You should be automatically redirected to a GitHub page asking for authorization: <code>Authorize application - my-new-cool-new-heroku-app by @foo-organization/foo-team - would like permission to access your account</code>. You can click on <code>Authorize application</code>.

----

Hope this helped. I ran into a [few problems which I discussed here](https://github.com/benbalter/jekyll-auth/issues/36).
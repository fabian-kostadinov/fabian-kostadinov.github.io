---
layout: post
title: Installation of Jekyll-Auth
tags: [jekyll-auth]
---
These are (hopefully) complete installation instructions for Jekyll-Auth.

__Attention:__ Before you try and install [Jekyll-Auth](https://github.com/benbalter/jekyll-auth), it is crucial to understand that Jekyll-Auth will _only_ work if you have a GitHub organization account which allows you to create teams. Having only a personal, (free or paid) account is _not_ enough for Jekyll to work.<!--more-->

----

_Step 1:_ Make sure you have a [Heroku account](http://www.heroku.com). A free one will be sufficient for most needs.

----

_Step 2:_ Make sure you have [Heroku Toolbelt](https://toolbelt.heroku.com/) installed. You will probably need to use your Heroku login information. When I first tried to <code>bundle install</code> Jekyll-Auth (see below), it worked, but I received this warning message:
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

_Step 3:_ If you've installed Heroku Toolbelt, you will probably have to recreate SSH keys, otherwise your local Heroku Toolbelt will not be able to push files to the remote server. Create a key <code>ssh-keygen -t rsa</code>, then add the key to Heroku <code>heroku keys:add</code>.

----

_Step 4:_ Use Heroku Toolbelt to [create a new Heroku app](https://devcenter.heroku.com/articles/creating-apps): <code>heroku create my-new-cool-heroku-app</code>. Your website will be available at <code>http://my-new-cool-heroku-app.herokuapp.com</code>, and it will have a git account to push to called <code>git@heroku.com:my-new-cool-heroku-app.git</code>.

----

_Step 5:_ The Heroku app will access the GitHub account to perform an authorization check for every user. If the user is registered with the corresponding GitHub account, she will also be allowed to access the Heroku app. Hence, the Heroku app must be registered with GitHub. Upon registration, you will receive a OAuth2 Client ID and Client Secret which will be needed at a later step.

Login to your GitHub account and go to <code>https://github.com/settings/applications/new</code>. Enter the following information:

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

_Step 7:_ Make sure you have [Ruby's bundler](http://bundler.io/) installed.


----

_Step 8:_ Create a local clone of the [Jekyll-Auth repository available on GitHub](https://github.com/benbalter/jekyll-auth).

----

_Step 9:_ Navigate to your local clone. There should be a <code>Gemfile</code> in your repository's directory. Change this file so that it looks like this:
{% highlight Ruby %}
source "https://rubygems.org"

gem 'jekyll-auth'
{% endhighlight %}

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
Here we need to enter the GitHub OAuth Client ID.

----

_Step 11e:_
{% highlight console %}
...
What's your GitHub Client Secret?
{% endhighlight %}
Then enter the GitHub Client Secret.

---

_Step 11f:_
{% highlight console %}
...
What's your GitHub Team ID?
{% endhighlight %}
Enter the GitHub Team ID. Be aware that you _cannot_ use a private (paid or unpaid) account's username, it _must_ be a team created with an organizational account.

----

_Step 12:_
Inside your local clone of Jekyll-Auth, create a file named <code>.env</code>. Put the following lines into this file providing your own OAuth2 Client ID, Client Secret and the GitHub Team ID.

{% highlight console %}
GITHUB_CLIENT_SECRET=abcdefghijklmnopqrstuvwxyz0123456789
GITHUB_CLIENT_ID=qwertyuiop0001
GITHUB_TEAM_ID=12345
{% endhighlight %}

----

_Step 13:_
Before we are ready to push our local clone of Jekyll-Auth to the remote Heroku server, we first need to add the Gemfile.lock to the repository:
{% highlight console %}
git add -f Gemfile.lock
git commit -m "Added Gemfile.lock"
{% endhighlight %}
Be aware that we use the <code>-f</code> parameter to enforce adding this file. If you do not provide the parameter git might refuse to add the file to the repository because it is actually ignored in <code>.gitignore</code>.

----

_Step 14:_
Now we are finally ready to push everything to the remote Heroku server: <code>git push heroku master</code>.

----

_Step 15:_
Open a browser and navigate to the Heroku URL <code>https://my-new-cool-herokuapp.herokuapp.com</code>. You should be automatically redirected to a GitHub page asking for authorization: <code>Authorize application - my-cool-new-heroku-app by @my-team-id would like permission to access your account</code>. You can click on <code>Authorize application</code>.

----

Hope this helped. I ran into a [few problems which I discussed here](https://github.com/benbalter/jekyll-auth/issues/36).
---
layout: post
title: How To Find A GitHub Team ID
tags: [github]
comments: true
---
In an earlier post I explained [how to install Jekyll-Auth]({% post_url 2014-11-13-installation-of-jekyll-auth %}). In GitHub, every team (and organization and user) receives a six to seven digits integer number as an ID like <code>1234567</code>. There are cases where you might need access to this information, for instance during the installation of [Jekyll-Auth](https://github.com/benbalter/jekyll-auth/). Unfortunately, there is no easy way to find out a team's ID. I could not find it anywhere published at the official GitHub website. You can however access this information through [the GitHub API](https://developer.github.com/v3/).<span class="more"></span>

The first thing you need to do is to make sure you have [the _cURL_ tool](http://curl.haxx.se/) installed. What is curl? To cite the website:
<blockquote>curl is a command line tool and library for transferring data with URL syntax, supporting [...] HTTP [...]</blockquote>
We will use cURL to send simple HTTP commands to the GitHub API. The response is a HTTP header followed by the requested information in a JSON format.

Once you have cURL installed, you first need to make sure that you are actually a member of the organization and team you want to find the ID for.

Next, you need to create a personal access token. When using the GitHub API you in most cases access protected information, thus we need to authorize first. When logged in to your GitHub account, go to -> *Settings* -> *Applications* -> (Personal access tokens) *Generate new token*. Provide a name. Select <code>read:org</code>, leave the others or unselect them. Click *Generate token*. Copy the token to your clipboard.

Open a shell. In the shell type by replacing the <code>&lt;very-long-access-token&gt;</code> with the actual access token and <code>&lt;org-name&gt;</code> with the organization's name. On Windows I had to use double quotes (") but maybe on Unix/Linux systems single quotes (') are presumed.
{% highlight shell-session %}
curl -H "Authorization: token <very-long-access-token>" https://api.github.com/orgs/<org-name>/teams

[
  {
    "name": "Owners",
    "id": 123456,
    "slug": "owners",
    "description": null,
    "permission": "admin",
    "url": "https://api.github.com/teams/123456",
    "members_url": "https://api.github.com/teams/123456/members{/member}",
    "repositories_url": "https://api.github.com/teams/123456/repos"
  },
  {
    "name": "my-team",
    "id": 1234567,
    "slug": "my-team",
    "description": "This is the team with the ID I am looking for",
    "permission": "push",
    "url": "https://api.github.com/teams/1234567",
    "members_url": "https://api.github.com/teams/1234567/members{/member}",
    "repositories_url": "https://api.github.com/teams/1234567/repos"
  },
  {
    "name": "Members",
    "id": 234567,
    "slug": "members",
    "description": null,
    "permission": "push",
    "url": "https://api.github.com/teams/234567",
    "members_url": "https://api.github.com/teams/234567/members{/member}",
    "repositories_url": "https://api.github.com/teams/234567/repos"
  }
]
{% endhighlight %}
Of course your output might differ a little bit, just look for the ID <code>"id": 1234567</code> in *my-team*.
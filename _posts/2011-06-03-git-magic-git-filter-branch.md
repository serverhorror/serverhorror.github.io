---
layout: post
title: Git Magic (git filter-branch)
categories: []
tags:
- Apache Subversion
- Author
- git
- Literature
- Revision control
- Source code
- Tools
status: publish
type: post
published: true
meta:
  reddit: a:2:{s:5:"count";s:1:"0";s:4:"time";s:10:"1322875041";}
  _wpas_skip_twitter: '1'
  _wpas_skip_fb: '1'
  _wpas_skip_linkedin: '1'
author: 
---
<p>Rewrite all commits in <code>git</code> to a new author:</p>
<p>{% highlight text %}
git filter-branch -f --env-filter '\
    GIT_AUTHOR_EMAIL=author@example.invalid; \
    export GIT_AUTHOR_EMAIL; \
    GIT_AUTHOR_NAME=&quot;New Author&quot;; \
    export GIT_AUTHOR_NAME'
{% endhighlight %}</p>

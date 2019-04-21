---
layout: post
title: "Setting up Disqus integration"
date: 2019-04-20
comments: true
---

This theme comes with [Disqus](http://disqus.com) integration.
All the Disqus Javascript and HTML code required to make Disqus work.
All you need to do is change the `disqus_shortname` field in `_config.yml`.
You can disable comments on a per-post level by adding the following line to YAML front matter
to the post:

```yaml
comments: false
```

To get a `disqus_shortname`, you'll need to [sign up](https://disqus.com/profile/signup/)
for a Disqus account. There is a delay (about 5-10 minutes) between setting up Disqus correctly
and seeing the comments and comment counts show up.
[This](http://www.perfectlyrandom.org/2014/06/29/adding-disqus-to-your-jekyll-powered-github-pages/)
outdated post describes this in more detail Disqus comments.

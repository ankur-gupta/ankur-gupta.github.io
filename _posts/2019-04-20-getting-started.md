---
layout: post
title: "Getting started"
date: 2019-04-20
comments: true
image: airport-coffee-cup-1684151-clipped.jpg
---

## First Steps

* Install [Jekyll](http://jekyllrb.com/docs/installation/)
* Download or clone [Laplacian](https://github.com/ankur-gupta/laplacian)
* Modify `_config.yml` as needed
* Test using [Jekyll](http://jekyllrb.com/docs/usage/)

```bash
jekyll serve --watch
```
## Add posts

The `permalink` field in `_config.yml` decides how the URL to posts are formed.
You can override the global `permalink` value in `_config.yml` by adding a `permalink`
field in the `.md` file.

If you're not ready to publish a post on your website, add it to `_drafts` which won't show
the post on the website but the file will be readable via the GitHub repository.

Also see [Trio](https://www.perfectlyrandom.org/trio/2015/09/06/getting-started-with-trio/)
for more details.
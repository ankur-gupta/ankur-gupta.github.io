---
layout: post
title: "Navigating Jekyll parsing"
update_date: 2014-09-19
comments: true
---

To productively work with Github Pages, one has to know their way around Liquid,
Markdown, Javascript and HTML.

### Liquid
If you're beginning with Jekyll, you need to look at the [Jekyll homepage](http://jekyllrb.com/).
The basics of Liquid can be found [Shopify](http://docs.shopify.com/themes/liquid-basics).
An extremely helpful yet quick summary is at
[Shopify's Github](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers).
Finally, when you get your hands through the sand, here is a
[cheatsheet](http://cheat.markdunkley.com/).

### Markdown
If you're completely new then you might want to begin with
[Markdown homepage](http://daringfireball.net/projects/markdown/) and
the [basic syntax](http://daringfireball.net/projects/markdown/syntax) to get some orientation.
But if you're one who dives into things first and reads the manual later, yo might like this
excellent, interactive, lightening-fast [tutorial](http://markdowntutorial.com/).
The [Github article](https://help.github.com/articles/markdown-basics) on Markdown basics is
also something to look at. There are many cheat sheets available, such as
[this one](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

If you're using [Sublime Text](http://www.sublimetext.com/3) as your text editor, then you
have access to many plugins that make your life easier. I use the
[Markdown Preview](https://github.com/revolunet/sublimetext-markdown-preview) package,
which you can easily install right from your Sublime Text's *Command Palette*. This plugin, once
installed, provides a cheatsheet in your Sublime Text editor via
*Command Palette > Open Markdown Cheat sheet*.

Note that there are many flavors of Markdown. I use *redcarpet* which was the default in the
[Tactile theme](https://github.com/jasonlong/tactile-theme). A very common one is the
[Github-flavored Markdown](https://help.github.com/articles/github-flavored-markdown).

### Javascript and HTML
I've got nothing. [W3Schools](http://www.w3schools.com/js/DEFAULT.asp) is where I learned
HTML, CSS and Javascript.

### Mixing them up!
Github Pages/Jekyll uses both HTML (`.html`) and Markdown(`.md`) pages. In both kind of pages,
Liquid gets a first pass at things. This means that if you have a Liquid tag, then it will be
interpreted first. Then, if it is a `.md` page, the Markdown will be converted to HTML
(including any javascript conversion) after Liquid tags are handled.

If you're wondering why this is important, then consider the following situation. You want to
add a Liquid snippet to your blog, which should not run
(see the raw markdown for this post as an example).

In your `_posts/blog-post-title.md` file, you cannot really add the snippet as is, even
when enclosed within triple back quotes like this
```
``` This is a Liquid tag ```
```
because the Liquid tag will be interpreted before the Markdown.
You need to use the Liquid's own quoting/escaping like this

```
{{ "{% raw " }}%}
These Liquid tags will not be run
{{ "{% endraw " }}%}
```

Another way to escape Liquid tags is
[here](http://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags). You also
might want to look at the [source code of this page](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/_posts/2014-06-29-getting-around-with-liquid-markdown-javascript-html.md).

Also note that, using an HTML comment `<!-- Liquid tag -->` will not serve as a substitute
for the Liquid quoting/escaping. As a result, you can't *simply* comment out a
Liquid tag using HTML comment construct.

The Liquid tag enclosed between by the HTML comment
construct will not prevent the Liquid engine from evaluating its Liquid tag.
If the Liquid tag evaluation succeeds, the result will be enclosed by the HTML comment
construct. If everything else works out as well, then the overall result will an
HTML comment. This HTML comment won't display on the web page but it will be present in the
source HTML and can be viewed if you view the source. As an example,

```html
{% raw %}
<!-- {% if page.url %} -->
<!-- Hello -->
<!-- {% endif %} -->
{% endraw %}
```

becomes
```html
<!--  -->
<!-- Hello -->
<!--  -->
```

in final HTML source. Another example,
```html
{% raw %}
<!-- {{ page.url }} -->
{% endraw %}
```

becomes
```html
<!-- /2014/06/29/getting-around-with-liquid-markdown-javascript-html/ -->
```

Yet another example, this alone, without the closing `endif` Liquid tag,
```html
{% raw %}
<!-- {% if page.url %} -->
{% endraw %}
```

will produce a Liquid error like this:
```
Liquid Exception: if tag was never closed in _posts/2014-06-29-getting-around-with-liquid-markdown-javascript-html.md
...error:
             Error: if tag was never closed
             Error: Run jekyll build --trace for more information.
```

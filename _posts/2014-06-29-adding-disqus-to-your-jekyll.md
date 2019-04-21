---
layout: post
title: "Adding Disqus to your Jekyll"
date: 2014-06-29
update_date: 2015-12-24
comments: true
---

I transferred my blog from a [Wordpress](http://old.perfectlyrandom.org) theme to Jekyll/GitHub Pages
because I liked the idea of writing up a post in [Markdown](http://daringfireball.net/projects/markdown/)
in my [favorite text editor](http://www.sublimetext.com/3) and then using [git](http://git-scm.com/)
to push the changes to [Github](https://github.com/). Add to that the speed and simplicity
of having a static (though [javascript](http://www.w3schools.com/js/DEFAULT.asp)-powered) website
*on* a free server. I don't have to log in to a (comparatively) slower, web browser based
interface of Wordpress hosted on a free server that runs [PHP](http://www.php.net/)
to dynamically create my [HTML](http://www.w3schools.com/html/DEFAULT.asp).

### Steps
1.  You will need to register your website with [Disqus](https://disqus.com/websites/).
2.  Get a customized version of the [Universal Embed code](https://disqus.com/admin/universalcode/)
    or edit the code to include the *unique identifier* of your Disqus account. The universal
    code is nothing but [Javascript](http://www.codecademy.com/en/tracks/javascript).
3.  You will need to add the code to either your HTML or Markdown files. Follow the instructions
    [here](https://help.disqus.com/customer/portal/articles/472138-jekyll-installation-instructions).

While these basic steps should do the trick, I did not get the desired behavior that easily.

First of all, when you are testing the page locally (instead of the live location), you need to
enable *developer mode* as follows
(thanks to [SchmidtHappens](http://schmidt-happens.com/articles/2011/09/26/adding-disqus-comments.html)
for that)
```javascript
var disqus_developer = 1; // Comment out when the site is live
```

Another helpful reference is [here](http://joshualande.com/jekyll-github-pages-poole/).

### Adding Disqus comments
I added the `disqus_thread` javascript code to every blog post.
This was easy and worked quite well. Here is a snippet from my
[`_layouts/post.html`](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/_layouts/post.html):

```javascript
{% raw %}
{% if page.comments %}
<div id="disqus_thread"></div>
    <script type="text/javascript">
        /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
        var disqus_shortname = 'perfectlyrandom'; // required: replace example with your forum shortname
        // var disqus_developer = 1; // Comment out when the site is live
        var disqus_identifier = "{{ page.url }}";

        /* * * DON'T EDIT BELOW THIS LINE * * */
        (function() {
            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
            dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
        })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
    <a href="http://disqus.com" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
{% endif %}
{% endraw %}
```

Note that the above javascript code uses `embed.js`.

I further *tightened* my Disqus integration following some tips from
[here](https://help.disqus.com/customer/portal/articles/565624-tightening-your-disqus-integration)
and defined a `discus_identifier` variable in my `disqus_thread` javascript code.
You can learn about other `disqus_` variables [here](https://help.disqus.com/customer/portal/articles/472098).

### Adding comment count
This is a trickier task than adding the comments. This is an example of the javascript
that I used (from [`_layouts/default.html`](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/_layouts/default.html)):
```javascript
<script type="text/javascript">
      /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
      var disqus_shortname = 'perfectlyrandom'; // required: replace example with your forum shortname
      // var disqus_developer = 1; // Comment out when the site is live

      /* * * DON'T EDIT BELOW THIS LINE * * */
      (function () {
        var s = document.createElement('script'); s.async = true;
        s.type = 'text/javascript';
        s.src = 'http://' + disqus_shortname + '.disqus.com/count.js';
        (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
      }());
    </script>
```

You need to add this `count.js` javascript code to every page on which you want to display the
number of comments. This can be the blog post itself (such as this page) or it can be the
list of posts (such as the [`index.html`](http://perfectlyrandom.org/index.html)).

Next, you need to add some HTML which puts the number of comments wherever you want it.
I wanted to show comment count on both the post and the index page.
This is the snippet from [`index.html`](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/index.html):
```html
{% raw%}
<a href="{{ post.url }}index.html#disqus_thread" data-disqus-identifier="{{post.url}}"></a>
{% endraw %}
```
and, this is the snippet from [`_includes/title-with-author.html`](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/_includes/title-with-author.html):
```html
{% raw %}
<a href="{{ page.url }}index.html#disqus_thread" data-disqus-identifier="{{page.url}}"></a>
{% endraw %}
```

The [`_includes/title-with-author.html`](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/_includes/title-with-author.html) file is included in [`_layouts/post.html`](https://github.com/ankur-gupta/ankur-gupta.github.io/blob/master/_layouts/post.html). The difference between the above two snippets is that one uses `post.url` and the
other uses `page.url`.

These steps seem like they would work, and they do. But there is one more thing that may help you
keep you from **setting your hair on fire**. Even when you do everything right, it takes Disqus
some time to handle the javascript (I have no idea why!). I noticed that the same exact code
would initially not show the comment count at all. But later on, after about 5-10 minutes and
after closing and reopening the web page, the comment count will magically appear. Similarly,
even after I wrote a comment to test things out, the web page would not show the correctly
updated count. I had to wait another few minutes to get the correct count. So, before you go
unnecessarily "correcting" your code, just wait and see if it works.

**Reminder:** Remember to change the `disqus_shortname` to your own forum shortname. If you keep on using my shortname, `perfectlyrandom`, as shown in the javascript above, I will get an email from Disqus about the comment.

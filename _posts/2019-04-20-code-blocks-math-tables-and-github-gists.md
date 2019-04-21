---
layout: post
title: "Code blocks, Math, Tables, and GitHub gists"
date: 2019-04-20
comments: false
---

This post is copied from [Trio](https://www.perfectlyrandom.org/trio/2015/09/06/code-blocks-tables-and-github-gists/)
and then modified with the aim to demonstrate some of the features.


## Headings
This is how the headings look like.

# Heading Level One (h1)
## Heading Level Two (h2)
### Heading Level Three (h3)
#### Heading Level Four (h4)
##### Heading Level Five (h5)
###### Heading Level Six (h6)


## Code blocks

Since Laplacian uses `redcarpet` instead of `kramdown`, we can use the triple backticks
to define fenced code blocks.

Here is how code blocks look in Laplacian:

```css
#container {
    float: left;
    margin: 0 -240px 0 0;
    width: 100%;
}
```

You can also use liquid tag `highlight` which has a similar effect:

{% highlight c %}
void main() {
    printf("Hello World!");
}
{% endhighlight %}

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

{% highlight css %}
#container {
    float: left;
    margin: 0 -240px 0 0;
    width: 100%;
}
{% endhighlight %}

{% highlight ruby %}
def what?
  42
end
{% endhighlight %}

You can also use the triple tilde ala `kramdown` which same the same effect:

~~~ ruby
def what?
  42
end
~~~

## MathJax Example
The [Einstein field equations](https://en.wikipedia.org/wiki/Einstein_field_equations) can be
displayed nicely as follows.

$$
R_{\mu\nu} - \frac{1}{2} R g_{\mu\nu} + \Lambda g_{\mu\nu} = \frac{8 \pi G}{c^4} T_{\mu\nu}
$$

Using original theme [Lagrange](https://lenpaul.github.io/Lagrange/)'s example, the [Schrödinger equation](https://en.wikipedia.org/wiki/Schr%C3%B6dinger_equation) looks like this:

$$
i\hbar\frac{\partial}{\partial t} \Psi(\mathbf{r},t) = \left [ \frac{-\hbar^2}{2\mu}\nabla^2 + V(\mathbf{r},t)\right ] \Psi(\mathbf{r},t)
$$

## Footnotes
Markdown footnotes[^1] work nicely in Laplacian. You need to make sure proper extensions are
enabled in either `redcarpet` or `kramdown` parsers.

[^1]: <http://en.wikipedia.org/wiki/Syntax_highlighting>

## Tables
This is how tables look. An excellent source to create tables in many formats (including Markdown)
is [Tables Generator](http://www.tablesgenerator.com/).

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |


## Github Gists
This is how GitHub Gists look in Laplacian.
<script src="https://gist.github.com/ankur-gupta/582bfba52054b9e8d9b3.js"></script>


## Blockquotes
Laplacian supports lists, `<hr>`s, `<table>`s  and

> blockquotes


---
layout: post
title: "Color highlighting for diff in ST3"
date: 2015-02-08
comments: true
image: "color-highlighting-diff-sublime/header.png"
---

I frequently need to perform a `diff` on two of my open views in Sublime Text 3. I use the [EasyDiff](https://github.com/facelessuser/EasyDiff) package for this purpose. This package works nicely and shows me the `diff` changes in a new ST3 tab. The only problem was that the changes showed up without color highlighting, even when I was using **Diff** syntax highlighting.

![No color highlighting using iPlastic theme](/assets/color-highlighting-diff-sublime/easydiff-st3-iplastic.png)

I was using iPlastic color theme that comes with ST3 by default. After searching for "diff color highlighting", I found a [few links](https://github.com/kemayo/sublime-text-git/issues/169) which claim to solve this issue. Somehow, these solutions did not work for me.

Then, I came across the following snippet of code in a color theme which did work:

```xml
<!-- Adding stuff for diff. This works in ST3 -->
        <dict>
            <key>name</key>
            <string>diff header from</string>
            <key>scope</key>
            <string>meta.diff.header.from-file</string>
            <key>settings</key>
            <dict>
                <key>background</key>
                <string>#FFDDDD</string>
                <key>fontStyle</key>
                <string></string>
                <key>foreground</key>
                <string>#000000</string>
            </dict>
        </dict>
        <dict>
            <key>name</key>
            <string>diff header to</string>
            <key>scope</key>
            <string>meta.diff.header.to-file</string>
            <key>settings</key>
            <dict>
                <key>background</key>
                <string>#DDFFDD</string>
                <key>fontStyle</key>
                <string></string>
                <key>foreground</key>
                <string>#000000</string>
            </dict>
        </dict>
        <dict>
            <key>name</key>
            <string>diff inserted</string>
            <key>scope</key>
            <string>markup.inserted.diff</string>
            <key>settings</key>
            <dict>
                <key>background</key>
                <string>#DDFFDD</string>
                <key>fontStyle</key>
                <string></string>
            </dict>
        </dict>
        <dict>
            <key>name</key>
            <string>diff deleted</string>
            <key>scope</key>
            <string>markup.deleted.diff</string>
            <key>settings</key>
            <dict>
                <key>background</key>
                <string>#FFDDDD</string>
                <key>fontStyle</key>
                <string></string>
                <key>foreground</key>
                <string>#000000</string>
            </dict>
        </dict>

```

I pasted this snippet to my `.tmTheme` file and `diff` color highlighting started to work.

In fact, I made a color theme called [GithubCleanWhite](http://colorsublime.com/theme/GitHubCleanWhite) which is similar to the Github color theme, has a white background, a grey unobtrusive color for comments, bold and italic font styles, a color for [BracketHighlighter](https://github.com/facelessuser/BracketHighlighter) to use and the above snippet for `diff` color highlighting. You can see a screenshot below:

![Color highlighting using GithubCleanWhite theme](/assets/color-highlighting-diff-sublime/easydiff-st3-githubcleanwhite.png)


You can take a better look on [Colorsublime](http://colorsublime.com/theme/GitHubCleanWhite) page. You can install it using the [Colorsublime](https://github.com/Colorsublime/Colorsublime-Plugin) package within ST3.


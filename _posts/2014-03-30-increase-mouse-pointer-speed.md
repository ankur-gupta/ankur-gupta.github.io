---
layout: post
title: Increase mouse pointer speed
date: 2014-03-30 04:31
author: Ankur Gupta
update_date: 2014-09-19
summary: Nobody likes a slow mouse. Learn how you can make it fast.
categories: [Lubuntu, lubuntu, mouse pointer speed, openbox, Tech]
logo: mouse-pointer
---


Very simple command as mentioned
[here](http://askubuntu.com/questions/27862/how-can-i-increase-the-mouse-pointer-speed-beyond-the-limits-set-by-the-mouse-pr).

Run this command in a terminal

```bash
xset m 3.5 3
```

The first number is acceleration (as in go `3.5x` faster). The second number is
threshold (number of pixels before accelerating).

You have to run this command every time you log in. So you might want to create a
script and add it to your start up scripts.




---
layout: post
title: Always on top in MacOS Yosemite
date: 2015-01-31
---

I love the fact that my terminal can stay on top of other windows, especially my text editor. See the beautiful screenshot below. 

![Terminal on top of text editor](/assets/terminal-on-top-of-text-editor.png)

Once upon a time, I managed to do this in MacOS Mavericks with an app called [Afloat](http://afloat.en.softonic.com/mac). This made me happy but my happiness was short-lived. 

On one of my Macs, I had to endure this problem: An annoying pop-up window with the words **"PlugSuit agent wants to make changes ... "** would pop up every hour and ask for my password. 

If you think this is not that annoying, imagine that you're in the middle of a Google hangout meeting and you're sharing your screen; suddenly out of nowhere, this window appears and demands your password. Until you successfully enter your password, it keeps on reappearing. No matter, you do this and move on with your life. An hour passes. You are now in deep programming mode. You are hammering away at your keyboard, bending the esoteric rules of computing to your will. Suddenly, this window reappears and insists on your password as a ransom to let you keep on working for another hour. I endured this, just so I can keep one window on top of the others.

Anyways, even this ability was taken away from me when I switched to Yosemite. 
I was never able to figure out why. 

_Goodbye Afloat_, I'll miss you!

I was frustrated. I even sent an email to Tim Cook about it. 

Fast forward a couple of weeks. In my guileless hopefulness, I google the words **"always on top in mac os"**. Apart from the usual links which I had previously, unsuccessfully, scoured for a solution, there was one new link -- a [Quora article](http://www.quora.com/Why-does-OS-X-not-have-always-on-top) on this issue. At the top of this page, there was a solution! 

My eyes widen with hope. Not only were there other people suffering the tyranny of Apple but that they had finally found a new solution that worked. Turns out that there is a Github repository named [Compulsion](https://github.com/alminde/Compulsion) that lets you do this. Sure, it requires me to build the Xcode project myself, but it claims to work. But there was a bigger problem -- [SIMBL](http://www.culater.net/software/SIMBL/SIMBL.php) was a requirement. I had previously tried to install and make SIMBL work. I never could do that in Yosemite (perhaps it was my fault). With a heavy heart, I google how to install SIMBL on Yosemite and I come across a Github repository called [EasySIMBL](https://github.com/norio-nomura/EasySIMBL) that purportedly makes installing SIMBL easy. 

<blockquote>
  <p>
    This is when the tide turned, the mountain gave way and the skies parted.
  </p>
</blockquote>

EasySIMBL is an incredibly well-thought out piece of code for various reasons:

1. Supports many OS X version: 10.7, 10.8, 10.9, 10.10
2. Reads plugins from `~/Library/Application Support/SIMBL/Plugins` only and never from `/Library/Application Support/SIMBL/Plugins`
4. Installing, upgrading and uninstalling were (almost) as easy as copying a folder! 

Talk about being easy.

I installed EasySIMBL and opened it. It turns out that Afloat was already listed as a plugin and was already enabled. I checked the "Use SIMBL" checkbox and quit SIMBL, just like the instructions said. 

![EasySIMBL window](/assets/easysimbl-window.png)

Then, I restarted my iTerm window. I clicked the "Window" menu item, and this is what I see

![Afloat Window Menu](/assets/afloat-window-menu.png)

This looks promising, but does it work?

I press the blessed key combination `Control + Command + A`. 

_Moment of truth._ 

![Afloat in action: Up](/assets/afloat-in-action-up.png)

It works! The terminal window stays on top of others. Jackpot! 

Let's see if everything works. I press the same key combination again and ...

![Afloat in action: Down](/assets/afloat-in-action-down.png)

... this works too. Mission accomplished. Commence blog writing. 

For the last 10 minutes, I have not experienced the extortionist demands for my password from the "PlugSuit agent wants to make changes ... " window. I will update this post if I run into that problem. 

So, people, your prayers (and mine) have been answered. You can put one window on top of others. Here is how to do it:

1. Install [EasySIMBL](https://github.com/norio-nomura/EasySIMBL)
2. Download Afloat [here](http://afloat.en.softonic.com/mac) or [here](http://www.macupdate.com/app/mac/22237/afloat). Install Afloat.
3. Enable SIMBL and Afloat in the EasySIMBL window. Quit EasySIMBL.
4. Enjoy.


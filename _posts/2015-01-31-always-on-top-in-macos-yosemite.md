---
layout: post
title: Always on top in MacOS Yosemite
date: 2015-01-31
update_date: 2015-12-24
comments: true
---
_**Update:** This works on El Capitan as well._

_As the next installment in the never-ending cat-and-mouse game between Apple and the developers, [EasySIMBL stopped working](https://github.com/norio-nomura/EasySIMBL/issues/25) for many people after updating to version 10.10.4 (read comments). As suggested by others, Afloat can still be made to work using [SIMBL 0.9.9](http://www.culater.net/software/SIMBL/SIMBL.php). **See the instructions at the end.**_


I love the fact that my terminal can stay on top of other windows, especially my text editor.
See the beautiful screenshot below.

![Terminal on top of text editor](/assets/always-on-top-in-macos-yosemite/terminal-on-top-of-text-editor.png)

Once upon a time, I managed to do this in MacOS Mavericks with an app called
[Afloat](http://afloat.en.softonic.com/mac). This made me happy but my happiness was short-lived.

On one of my Macs, I had to endure this
[problem](http://infinite-labs.net/kb/plugsuit/ps-remove-on-10.6.html):
An annoying pop-up window with the words **"PlugSuit agent wants to make changes ... "**
would pop up every hour and ask for my password.

If you think this is not that annoying, imagine that you're in the middle of a Google hangout
meeting and you're sharing your screen; suddenly out of nowhere, this window appears and demands
your password. Until you successfully enter your password, it keeps on reappearing. No matter,
you do this and move on with your life. An hour passes. You are now in deep programming mode.
You are hammering away at your keyboard, bending the esoteric rules of computing to your will.
Suddenly, this window reappears and insists on your password as a ransom to let you keep on
working for another hour. I endured this, just so I can keep one window on top of the others.

Anyways, even this ability was taken away from me when I switched to Yosemite.
I was never able to figure out why.

_Goodbye Afloat_, I'll miss you!

I was frustrated. I even sent an email to Tim Cook about it.

Fast forward a couple of weeks. In my guileless hopefulness, I google the words
**"always on top in mac os"**. Apart from the usual links which I had previously, unsuccessfully,
scoured for a solution, there was one new link --
a [Quora article](http://www.quora.com/Why-does-OS-X-not-have-always-on-top) on this issue.
At the top of this page, there was a solution!

My eyes widen with hope. Not only were there other people suffering the tyranny of Apple but
that they had finally found a new solution that worked. Turns out that there is a Github
repository named [Compulsion](https://github.com/alminde/Compulsion) that lets you do this.
Sure, it requires me to build the Xcode project myself, but it claims to work. But there was a
bigger problem -- [SIMBL](http://www.culater.net/software/SIMBL/SIMBL.php) was a requirement.
I had previously tried to install and make SIMBL work. I never could do that in
Yosemite (perhaps it was my fault). With a heavy heart, I google how to install SIMBL on
Yosemite and I come across a Github repository called
[EasySIMBL](https://github.com/norio-nomura/EasySIMBL) that purportedly makes installing
SIMBL easy.

<blockquote>
  <p>
    This is when the tide turned, the mountain gave way and the skies parted.
  </p>
</blockquote>

EasySIMBL is an incredibly well-thought out piece of code for various reasons:

1. Supports many OS X version: 10.7, 10.8, 10.9, 10.10
2. Reads plugins from `~/Library/Application Support/SIMBL/Plugins` only and never
from `/Library/Application Support/SIMBL/Plugins`
4. Installing, upgrading and uninstalling were (almost) as easy as copying a folder!

Talk about being easy.

I installed EasySIMBL and opened it. It turns out that Afloat was already listed as a plugin
and was already enabled. I checked the "Use SIMBL" checkbox and quit SIMBL, just like the
instructions said.

![EasySIMBL window](/assets/always-on-top-in-macos-yosemite/easysimbl-window.png)

Then, I restarted my iTerm window. I clicked the "Window" menu item, and this is what I see

![Afloat Window Menu](/assets/always-on-top-in-macos-yosemite/afloat-window-menu.png)

This looks promising, but does it work?

I press the blessed key combination `Control + Command + A`.

_Moment of truth._

![Afloat in action: Up](/assets/always-on-top-in-macos-yosemite/afloat-in-action-up.png)

It works! The terminal window stays on top of others. Jackpot!

Let's see if everything works. I press the same key combination again and ...

![Afloat in action: Down](/assets/always-on-top-in-macos-yosemite/afloat-in-action-down.png)

... this works too. Mission accomplished. Commence blog writing.

For one entire day, I have not experienced the extortionist demands for my password
from the _"PlugSuit agent wants to make changes ... "_ window. I will update this post if
I run into that problem.

So, people, your prayers (and mine) have been answered. You can put one window on
top of others. Here is how to do it:

**Original instructions**
_(these probably won't work on OS 10.10.4; read July 18, 2015 update below)_

1. Install [EasySIMBL](https://github.com/norio-nomura/EasySIMBL)
2. Download Afloat [here](http://afloat.en.softonic.com/mac) or
[here](http://www.macupdate.com/app/mac/22237/afloat). Install Afloat.
3. Enable SIMBL and Afloat in the EasySIMBL window. Quit EasySIMBL.
4. Enjoy.

#### Update
(June 18, 2015)

As of today, the instructions above have worked very well for me. Afloat has been working nicely and silently. Here are some updates on issues people have faced and graciously reported:

1. If you regularly see the pop-up window that says _Donate for Afloat_, try clicking on _Show Application_ button to go to the plugins folder and delete the file _Donate for Afloat.app_. If this doesn't work, try out the suggestions by Matt and Drew below in the comments.

2. If you're willing to install SIMBL (though EasySIMBL could also potentially work) and then build the Afloat project in Xcode, try [this link](https://github.com/rinckd/afloat), as suggested by Paul Irish in the comments section. You can find an active community and issues [here](https://github.com/millenomi/afloat/issues). The benefit of building Afloat yourself is that the source code is maintained and has better compatibility with Yosemite. I have not tried out this approach myself because the downloaded binary file seems to work very nicely (jinx).

#### Update
(July 18, 2015)

As of today, EasySIMBL stopped working on my Mac OS Yosemite 10.10.4. Surprisingly, I have another Mac, also Yosemite 10.10.4 with the same upgrades, on which EasySIMBL and Afloat are still working. EasySIMBL's problems with 10.10.4 have been documented [here](https://github.com/norio-nomura/EasySIMBL/issues/25) as pointed out by *cdz* in the comments. Fortunately, Afloat can still be made to work with SIMBL (instead of EasySIMBL) with very little effort (thanks to *cdz*!). Here are the instructions:

**Copy Afloat and uninstall EasySIMBL**

1. Open EasySIMBL. You can use `Command+Space` and type in `EasySIMBL` or open it from `~/Applications` or `/Applications`.
2. Before we uninstall EasySIMBL, let's copy `Afloat.bundle` to somewhere else. You can click on `Show Plugins Folder` and copy the folder `Afloat.bundle` to somewhere you can get to later. This folder is all you need to get Afloat working.
3. In EasySIMBL window, disable Afloat plugin and disable `Use SIMBL`.
4. Remove the folder `EasySIMBL.app` from either `~/Applications` or `/Applications`. See [EasySIMBL github page](https://github.com/norio-nomura/EasySIMBL) to for reference.
5. You may want to reboot (not strictly necessary but it helps with exorcism of ghosts).

**Install SIMBL**

1. Download [SIMBL 0.9.9](http://www.culater.net/software/SIMBL/SIMBL.php).
2. Extract the downloaded archive and run the `.pkg` installer file. You may need to go to `System Preferences > Security & Privacy` to allow running the installer.
3. Finish installation of SIMBL.

**Enable Afloat as a SIMBL plugin**

1. If you already have the `Afloat.bundle` folder, then simply copy/paste this folder into `/Library/Application Support/SIMBL/Plugins`.
2. If you don't have `Afloat.bundle`, you have two choices:

    a. Either build Afloat yourself as mentioned in the June 18, 2015 update above.

    b. Follow the original instructions described in this page that use EasySIMBL. You may choose to simply install Afloat and search for the folder `Afloat.bundle`, instead. Once you obtain this folder, you can most likely uninstall Afloat.
3. Quit and restart windows that you want Afloat to work on.

#### Update
(December 24, 2015)

Rocky Wu has created a [script](https://github.com/rwu823/afloat) to install Afloat quickly. If you're looking for a quick solution, you might want to give that a try. Many people have used it successfully.





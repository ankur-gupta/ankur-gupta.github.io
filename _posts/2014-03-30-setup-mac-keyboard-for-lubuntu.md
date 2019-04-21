---
layout: post
title: "Using a Mac keyboard in Lubuntu"
author: "Ankur Gupta"
date: 2014-03-30
categories: [change shortcuts, keyboard, Lubuntu, lubuntu, mac, openbox, Tech]
comments: true
---

I have recently made the switch from Windows laptop/Lubuntu desktop to a
Macbook Pro laptop/Lubuntu desktop. This unfortunately meant that I had to get
used to the Mac keyboard designs.

I bought a wired Mac keyboard to code on my Macbook Pro. It's very difficult to modify my
habits (and muscle memory) to handle both Windows/Ubuntu and Mac style keyboards.
So, I have decided to switch all my keyboard shortcuts in Lubuntu to behave like a Mac.
This way I can efficiently use my wired Mac keyboard with Lubuntu.

This is a very slow and painful process. I have to scrounge bits of information
from all over the web to get a fully tricked-out, working Mac keyboard. I intend to
list all steps I had to take in one place for someone else (and for myself in the
future when I upgrade/re-install computers).

Here are the steps I took.

1. Add Keyboard Layout Handler to the *LXPanel*. Right click to access the
*Keyboard Layout Handler Settings*. Change the keyboard type to the correct Apple keyboard.
I chose `applealu_ansi` because I have the *wired, aluminium Apple keyboard of the US variety*
(as opposed to the European version). This will ensure that the keystrokes will at
least be recognized. But, they won't all do as you want them to. You now need to edit
the shortcuts in many many places to ensure the results you want. Remember the
`Command` key is treated as the `Windows` or `Super` key.

2. Edit `~/.config/openbox/lubuntu-rc.xml` again!

Let's start with `Alt+Tab` and `Alt+Shift+Tab` to `Command+Tab` and
`Command+Shift+Tab` respectively. Change this section

```xml
<keybind key="A-Tab">
      <action name="NextWindow">
        <dialog>icons</dialog>
        <finalactions>
          <action name="Focus"/>
          <action name="Raise"/>
          <action name="Unshade"/>
        </finalactions>
      </action>
    </keybind>
    <keybind key="A-S-Tab">
      <action name="PreviousWindow">
        <finalactions>
          <action name="Focus"/>
          <action name="Raise"/>
          <action name="Unshade"/>
        </finalactions>
      </action>
    </keybind>
```

to

```xml
<keybind key="W-Tab">
      <action name="NextWindow">
        <dialog>icons</dialog>
        <finalactions>
          <action name="Focus"/>
          <action name="Raise"/>
          <action name="Unshade"/>
        </finalactions>
      </action>
    </keybind>
    <keybind key="W-S-Tab">
      <action name="PreviousWindow">
        <finalactions>
          <action name="Focus"/>
          <action name="Raise"/>
          <action name="Unshade"/>
        </finalactions>
      </action>
    </keybind>
```

by replacing `A (Alt)` to `W (Command)`.

There are plenty more things you can do. You can download my `lubuntu-rc.xml`
file and look through it yourself. On general principle, you should make a copy of
your `lubuntu-rc.xml` file before you edit it or overwrite it.

3.  Now, we come to the text editors. Remember when I said this is a slow and painful process.
That's because you will need to edit the keyboard shortcuts for every application
separately. I use [Sublime Text 3](http://www.sublimetext.com). You can download
this `Default (Linux).sublime-keymap` file that has the shortcuts I use. You can add the
contents of this file to your `~/.config/sublime-text-3/Packages/User/Default (Linux).sublime-keymap`
file. You will need to take care of repetitions. If you don't have one, then you
can create one. In fact, you could start with my file and edit/add
more shortcuts later on. Take a look at default shortcuts under
*Preferences* > *Key bindings--Default*.

You may not like all the shortcuts that I have in the file. But it's always worth it to look
over the file to familiarize yourself with your text editor shortcuts.


4. You should feel your sublime working quite smoothly after the above changes.
But now comes the big problem. There will always be some applications which do not
allow you to change key bindings. Google Chrome is a stunning example of this. Though you
can install [Shortcut Manager extension](https://chrome.google.com/webstore/detail/shortcut-manager/mgjjeipcdnnjhgodgjpfkffcejoljijf)
and try to change a few keyboard shortcuts. This did not work for me. Sadly, the
Shortcut Manager does not allow me to change the keyboard shortcut for
copying and pasting (even though it looks like it does!).
As an alternative you can make a system wide binding using `autokey`.

First, install `autokey`:
```bash
sudo apt-get install autokey-gtk
```

Run `autokey` by typing `autokey` in a terminal. You can find a beginner's guide
to `autokey` [here](https://code.google.com/p/autokey/wiki/BeginnersGuide).

You can create a new folder for your scripts (by clicking on the *New* button).
I recommend `~/bin/autokey` as the folder to keep all your files. Once you create
the folder, you can now add *scripts* and *phrases* to this folder.

Let's take the most used keyboard shortcut -- `Control+C` for copying.
**This method will not replace `Control+C` by `Command+C`**.
Instead, we will bind the `Command+C` to `Control+C`. This way both these keyboard
shortcuts will do the same thing -- *copying*. This is exactly why this method is
 not my favorite. It essentially renders both Control and Command equivalent.
 I recommend the first method of changing Openbox configuration in `lubuntu-rc.xml`
 as the **best**. Next best method is to change key bindings in the
 application itself (like we did for Sublime Text 3). Finally, since we have no
 other options, we have to use `autokey` for the remaining shortcut keys.

There are two ways you can do this -- scripts and phrases.
Both can bind the key combinations we want in the same way. You can choose
whichever one you like. Phrases might feel a tad simpler. I describe both methods below.

### Scripts
Click on *New > Script*. Give the script a name such as *copy*. In the big text box, replace
```python
# Enter script code
```

by
```python
keyboard.send_keys("<ctrl>+c")
```

Then, set the *Hotkey* as `Super+c`. Save the script by clicking on *Save*. Saving the script will create a file `copy.txt` in the folder location you chose. Repeat this method for many other shortcuts. You can take a look or download my autokey folder here. <br/>

### Phrases
Click on *New* > *Phrase*. Give the phrase a name such as `copy`. In the big text box, replace
```python
Enter phrase contents
```

by
```python
<ctrl>+c
```

Then, set the *Hotkey* as `Super+c`. Save the phrase by clicking on *Save*.
Saving the phrase will create a file `copy.txt` in the folder location you chose.
Repeat this method for many other shortcuts. You can take a look or download my
autokey folder [here](https://github.com/ankur-gupta/bin).

Obviously, you only need to create a script or a phrase for a key binding.
Creating both a script and phrase with the same name will most probably overwrite the file.

You can test the new keyboard shortcuts. As long as `autokey` is running,
the scripts and phrases will work. All we now need to do is make sure `autokey`
runs at startup. You can add `autokey` to your
[list of startup programs]({% post_url 2014-03-30-setup-autostart-in-lubuntu %}).

*To be continued ...*

---
layout: post
title: "Try out the fish shell"
date: 2014-09-21
update_date: 2014-09-22
categories: fish shell bash
comments: true
image: "try-out-fish-shell/header.png"
---

[Fish](http://fishshell.com/) is a new shell which makes using the command line much easier. If you've already got a very heavily configured traditional shell (like _bash_) then you probably have replacements for many features of fish. With fish, you get these features out of the box (for the most part).

There are many excellent posts and documentation on the web about installing, setting up and using the fish shell. On this page, my aim is to give you a quick overview of the most important features of fish so that you can make up your mind. I have also provided links to the most useful posts which might save you some time.

## Features of fish
1.  **fish is not bash**

    Before we even begin, let's note that fish and bash are different, just like _csh_ and bash are different.
    Many commands and constructs in fish work the same way as bash but there some commands that don't work the same way. One such example is setting the value of a variable. These are the ways you can add your personal `bin` folder to path in bash and fish, respectively
    ```bash
    # bash
    export PATH=~/bin:${PATH}
    ```

    ```bash
    # fish
    set -g -x PATH ~/bin $PATH
    ```

    Fish and bash are different enough that you may not be able to use some bash designed utilities directly. Instead, you would probably need fish-equivalents of those utilites. One such an example is the [bashmarks](https://github.com/huyng/bashmarks) utility which helps you jump between folders in bash. With fish, you can install the fish-equivalent, [fishmarks](https://github.com/techwizrd/fishmarks) instead.

2. **Colorful, underlined prompt and smarter completions**

    The most prominent feature of fish is the colorful, underlined prompt which provides more visual feedback than bash. For example, if a folder is present, you see the entire name of the folder even before you press a `Tab` key. The already typed letters appear as underlined, <span style="color:cyan;text-decoration:underline">cyan</span> colored and the rest of the untyped letters appear in <span style="color:grey">grey</span>.

    ![Fish helps you find existent folders](/assets/try-out-fish-shell/fish-underlined-commands.png)

    If the letters you've typed do not correspond to any matching completion, you see the letters in <span style="color:red">red</span>. Again, all of this happens before you press the `Tab` key.

    ![Fish tells you when there is no matching completion](/assets/try-out-fish-shell/fish-error-red.png)

    You can find more examples [here](http://fishshell.com/assets/img/screenshots/autosuggestion.png) and [here](http://fishshell.com/assets/img/screenshots/colors.png).

    Another cool feature is _&#8594;-key completion_ in which pressing the &#8594; arrow key completes the current completion. This completion is different than bash's `Tab`-completion, which does not perform completion when the completion is non-unique.
    Bash's `Tab`-completion instead provides the user with all possible completions. The user, then, has to type enough letters to make the completion unique. Fish's &#8594;-key completion works even when completion is non-unique. I recommend you try it out to see how good it feels.

    Further, fish's `Tab`-completion is different than bash's. Fish's `Tab`-completion shows all possible completions like bash, but it goes a step further. Repeatedly pressing the `Tab` key cycles through all the possible completions, thus eliminating the need to type in more characters.

    Another important completion in fish is the _switch-completion_, which is absent in bash. Fish's auto-completion looks through the switches of a command and completes them for you. For example, in bash, typing `tree --` and pressing `Tab` will not yield any results.

    ```bash
    # bash does not provide switch-completion
    tree --
    ```

     With fish, switches are completed just like the files and folders without even pressing the `Tab` key.

    ![Fish switch completion](/assets/try-out-fish-shell/fish-switch-completion.png)

3. **Integration with git**

    You may already know how to [change your bash command prompt](http://blog.taylormcgann.com/tag/prompt-color/) to show your _git branch_ and _git status_. With fish, you can get this functionality and many other features by installing [tacklebox](https://github.com/justinmayer/tacklebox).[^1]

    Tacklebox allows you to easily install community-scrutinized themes, plugins and modules which usually work out-of-the-box. I used the `urdh` theme, which shows git information in the following manner:

    ![Fish theme showing git information](/assets/try-out-fish-shell/fish-git-status-indicator.png)

    Note how the <span style="color:green">_green check mark_</span> changes into <span style="color:orange">_orange solid dot_</span> when the git repository status changes from _clean_ to _unstaged_.

    See another example [here](http://fishshell.com/assets/img/screenshots/works_out_of_the_box.png).

    [^1]: Get it ? [Fish](http://en.wikipedia.org/wiki/Fish), [tacklebox](http://en.wikipedia.org/wiki/Fishing_tackle#Tackle_boxes) ... ?

4. **fish-config web interface**

    This is the most unorthodox feature of the fish shell. You can change colors and prompts, look at your functions and variables and even modify your command history using the web interface. This may not appeal to those who love the command line but it's very handy should you need it. Type this on the fish prompt

    ```bash
    fish_config
    ```

    and it will open up local webpage like this one

    ![Fish config web interface](/assets/try-out-fish-shell/fish-config-web-interface.png)

    Another example [here](http://fishshell.com/assets/img/screenshots/web_config.png).


I am sure there are many more features of fish which I have not yet discovered yet.

## Installing fish
1. Make sure you get fish v2.1.0 or later. Fish v2.0.0 does not have the
    latest `Tab`-completion tricks that I described.

    For Mac OSX, the post on [Hacker Codex](http://hackercodex.com/guide/install-fish-shell-mac-ubuntu/) is the best page I found for installing and setting up fish.

    For Ubuntu, you can dowload the `.deb` [here](http://fishshell.com/) and then use the following command

      ```bash
      sudo dpkg -i <fish-deb-file>
      ```

2. The [fish official tutorial](http://fishshell.com/docs/current/tutorial.html) is a good reference for customizing fish to your taste. The documentation is [here](http://fishshell.com/docs/current/index.html).
3. You probably would want to install [Tacklebox](https://github.com/justinmayer/tacklebox) which automatically installs [Tackle](https://github.com/justinmayer/tackle).
4. Install [fishmarks](https://github.com/techwizrd/fishmarks).


## Some tips and tricks and snippets
1.  Some plugins of fish may have dependencies that you may need to install.
    For example, if you find your fish prompt complaining about something called `vcprompt`, you need to install it.

    Funnily enough, it's easier to install `vcprompt` on Mac OSX then on ubuntu.

    ```bash
    # On Mac OSX
    brew install vcprompt
    ```

    On Ubuntu, you will need to manually install `vcprompt` by building it from source. [These](http://choorucode.com/2014/05/22/vcprompt/) commands worked for me

    ```bash
    # On ubuntu
    hg clone https://bitbucket.org/gward/vcprompt
    cd vcprompt
    autoconf
    ./configure
    make
    sudo make install
    ```


    Same goes for `grc`. Simiarly, if you have a [mercurial](http://mercurial.selenic.com/) repository, you might need to install [`hg-prompt`](http://sjl.bitbucket.org/hg-prompt/installation/). [These](http://sjl.bitbucket.org/hg-prompt/installation/) instructions worked for me on both Mac OSX and on Ubuntu.

2. Set the `urdh` theme by adding this to your `~/config/fish/config.fish` file

    ```bash
    set tacklebox_theme urdh
    ```

3. Some simple snippets to get you started.
   Put these in your `~/config/fish/config.fish` file.
    ```bash
    function reload
        source ~/.config/fish/config.fish
    end

    function subl
        open -a "Sublime Text.app" $argv
    end

    function ll
        ls -lhG $argv
    end

    function la
        ls -lahG $argv
    end

    function lsd
        ls -d */
    end
    ```

4. Redefining a command.
   Fish doesn't have your traditional bash `alias` command. Instead you need to define a function. The following fish function, however, has infinite recursion which will give you an error
     ```bash
     # Inifinite recursion: this will give you an error
     function ls
          ls -hG $argv
     end
     ```

     Instead you can get the `alias`-like functionality by writing this instead

     ```bash
     # These should work
     function ls
          command ls -hG $argv
     end

     function grep
          command grep --color=auto $argv
     end
     ```

Finally, if you know of an excellent use of fish that I've missed, please let me know in the comments.

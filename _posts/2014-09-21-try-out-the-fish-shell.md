---
layout:     post
title:      Try out the fish shell
date:       2014-09-21
summary:    Fish is a new shell with tons of cool features.
categories: fish shell bash
---

[Fish](http://fishshell.com/) is a new shell which makes using the command line much easier. To be fair, if you've already got a very heavily configured traditional shell (like _bash_) then you probably have some sort of replacement for many of the features of fish. What makes fish different than bash is that you get those features out of the box (for the most part). 

There are many excellent posts and documentation about installing, setting up and using the fish shell on the web. On this page, my aim is to give you a quick overview of the most important features of fish so that you can make up your mind. I also provide the links to the most useful posts which might save you some time. 

### Features of fish

1.  **fish is not bash**

    Before we even begin, let's note that fish and bash are different, just like _csh_ and bash are different. 
    Many commands and constructs in fish work the same way as bash and there are a few that don't. For example, these are the ways you add your personal `bin` to path in bash and fish 

    ```bash
    # bash
    export PATH=~/bin:${PATH}
    ```

    ```bash
    # fish
    set -g -x PATH ~/bin $PATH
    ```

    Fish and bash are so different that you would probably need fish-equivalents of some utilites that you've been using in bash. Such an example is [bashmarks](https://github.com/huyng/bashmarks). You will need to install the fish-equivalent [fishmarks](https://github.com/techwizrd/fishmarks).

2. **Colorful, underlined prompt and smarter completions**

    The most prominent feature of fish is the colorful, underlined prompt which provides more visual feedback than bash. For example, if a folder is present, you see the entire name of the folder even before you press a `Tab`key. The letters you have typed appear as <span style="color:cyan">_cyan_</span> colored and underlined and the rest of the folder name appears in <span style="color:grey">_grey_</span>. 

    ![Fish helps you find existent folders](/assets/fish-underlined-commands.png)

    Another cool feature is that you can press the `Forward` arrow key completes the shown current completion. This is different than what the `Tab` key does. You might need to experience this for yourself to understand it. 

    If the letters you've typed do not correspond to any matching completion, you see the letters in <span style="color:red">_red_</span>. Again, all of this happens before you press the `Tab` key.

    ![Fish tells you when there is no matching completion](/assets/fish-error-red.png)

    You can find more examples [here](http://fishshell.com/assets/img/screenshots/autosuggestion.png) and [here](http://fishshell.com/assets/img/screenshots/colors.png).

    Another important completion which I found lacking in bash was the _switches_. For example, in bash, typing `tree --` and pressing `Tab` will not yield any results. With fish, you don't even have to press `Tab` to see the completions

    ![Fish switch completion](/assets/fish-switch-completion.png)

3. **Integration with git**
  
    You may already know how to [change your bash command prompt](http://blog.taylormcgann.com/tag/prompt-color/) to show your _git branch_ and _git status_. With fish, you can get this functionality and many other features by installing [tacklebox](https://github.com/justinmayer/tacklebox).[^1]

    Tacklebox allows you to easily install community-scrutinized themes, plugins and modules which usually work out-of-the-box. I used the `urdh` theme, which shows git information in the following manner:

    ![Fish theme showing git information](/assets/fish-git-status-indicator.png)

    Note how the <span style="color:green">_green check mark_</span> changes into <span style="color:orange">_orange solid dot_</span> when the git repository status changes from _clean_ to _unstaged_. 

    See another example [here](http://fishshell.com/assets/img/screenshots/works_out_of_the_box.png).

    [^1]: Get it ? [Fish](http://en.wikipedia.org/wiki/Fish), [tacklebox](http://en.wikipedia.org/wiki/Fishing_tackle#Tackle_boxes) ... ?

4. **fish-config web interface**

    This is the most unorthodox feature of the fish shell. You can change colors and prompts, look at your functions and variables and even modify your command history using the web interface. This may not appeal to those who like the command line but it's very handy should you need it. Type this on the fish prompt

    ```bash
    fish_config
    ```

    and it will open up local webpage like this one

    ![Fish config web interface](/assets/fish-config-web-interface.png)

    Another example [here](http://fishshell.com/assets/img/screenshots/web_config.png). 


I am sure there are many more features of fish which I have not yet discovered yet. 

### Installing fish

1. The post on [Hacker Codex](http://hackercodex.com/guide/install-fish-shell- mac-ubuntu/) is the best page I found for installing and setting up fish. 
2. The [fish official tutorial](http://fishshell.com/docs/current/tutorial.html) is a good reference for customizing fish to your taste. The documentation is [here](http://fishshell.com/docs/current/index.html). 
3. You probably would want to install [Tacklebox](https://github.com/justinmayer/tacklebox) which automatically installs [Tackle](https://github.com/justinmayer/tackle).
4. Install [fishmarks](https://github.com/techwizrd/fishmarks).


### Some tips and tricks and snippets

1.  Some plugins of fish may have dependencies that you may need to install.
    For example, if you find your fish prompt complaining about something called `vcprompt`, you need to install it 

    ```bash
    # On Mac OSX
    brew install vcprompt
    ```

    Same goes for `grc`.

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

<br/><br/>
Finally, if you know of an excellent use of fish that I've missed, please let me know in the comments.













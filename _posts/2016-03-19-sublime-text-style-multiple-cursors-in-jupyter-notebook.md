---
layout: post
title: "Sublime-style multiple cursors in Jupyter"
date: 2016-03-19
update_date: 2016-03-23
comments: true
image: "sublime-style-multiple-cursors-in-jupyter/jupyter-notebook-multiple-cursors-demo.gif"
---

## Jupyter Notebooks

[Jupyter Notebooks](http://jupyter.org/) are great for visualizing and sharing results with others.

![Jupyter Notebook running Python](/assets/sublime-style-multiple-cursors-in-jupyter/jupyter-screenshot.png)

Learning the keyboard shortcuts tremendously improves productivity while using Jupyter Notebooks. By default, you can see the keyboard shortcuts help dialog window by first pressing `Escape` to enter the `Command Mode` and then pressing `h`.

![Jupyter Notebook Keyboard Shortcuts Help Dialog Window](/assets/sublime-style-multiple-cursors-in-jupyter/jupyter-keyboard-shortcuts.png)

While these keyboard shortcuts are very useful, I miss the
[multiple cursor functionality from Sublime Text](https://www.sublimetext.com/). This functionality allows you to select (and then edit) multiple instances of a word visually. This is incredibly useful while coding. Renaming a variable via multiple cursors is easy, safe, and very satisfying. See the [Sublime Text
homepage](https://www.sublimetext.com/) or [see below](#multiple-cursors-demo) for an animated demo.

A Python expert informed me that this functionality was added to Jupyter Notebooks after switching to [Code Mirror 4](https://codemirror.net/) but it requires setting up a Sublime Text keymap. After searching a bit for instructions to accomplish this task, I found two GitHub issues
([this](https://github.com/jupyter/notebook/issues/1006) and
[this](https://github.com/ipython/ipython/pull/6221#issuecomment-58936367))  that describe how to do this. After some tinkering, I was able to make this work. I have consolidated these instructions below so it's easier for others.

_(Update: Excitingly, there is an [attempt](https://github.com/jupyter/notebook/pull/1109) in progress to allow the users to switch keymaps using the "Edit" menu. Until this pull request gets merged and rolled out in a new release, the following instructions would be helpful.)_

Overall, this task is simple. To set up a Sublime Text keymap, you need to add a few lines of Javascript to a `custom.js` file. Setting up a `custom.js` file for Jupyter Notebook is described in detail [here](https://jupyter-notebook.readthedocs.org/en/latest/examples/Notebook/rstversions/JavaScript%20Notebook%20Extensions.html).

## Instructions

1. Find the location of `custom.js` file.
    On MacOS and Linux, the default location is `~/.jupyter/custom/custom.js`. If this is your first time setting up a `custom.js`, this file will probably not exist in that location. Optionally, you can run the following code in a Jupyter Python Notebook to find the location and contents of your `custom.js` file :

    ```py
    # Print the location of Jupyter's config directory
    from jupyter_core.paths import jupyter_config_dir
    jupyter_dir = jupyter_config_dir()
    print(jupyter_dir)

    # Print the location of custom.js
    import os.path
    custom_js_path = os.path.join(jupyter_dir, 'custom', 'custom.js')
    print(custom_js_path)

    # Print the contents of custom.js, if it exists.
    if os.path.isfile(custom_js_path):
        with open(custom_js_path) as f:
            print(f.read())
    else:
        print("You don't have a custom.js file")
    ```


2. If `custom.js` does not exist at the location found in Step 1, create it.
    As a quick (optional) check that the `custom.js` indeed has an effect, add the following line to it (preferably at the top of the file):

    ```js
    alert("hello world from custom.js")
    ```

    Restart Jupyter Notebook server (by pressing `Control+C` in the terminal, if applicable). If everything is working correctly, you should be greeted by a dialog window in your browser after you restart your Jupyter Notebook.


3. You can comment out the line you added in Step 2. Add the following lines to your `custom.js` (preferably at the top of the file):

    ```js
    require(["codemirror/keymap/sublime", "notebook/js/cell"], function(sublime_keymap, cell) {
        cell.Cell.options_default.cm_config.keyMap = 'sublime';
    });
    ```

    Restart Jupyter Notebook again.

4. Test out multiple cursors by selecting some text and then pressing `Control+D` (on Linux or Windows) or `Command+D` (on MacOS).

![Multiple cursors in Jupyter Notebook](/assets/sublime-style-multiple-cursors-in-jupyter/jupyter-notebook-multiple-cursors-demo.gif)


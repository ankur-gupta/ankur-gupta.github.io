---
layout: post
title: "Aero snap using keyboard"
author: "Ankur Gupta"
date: "2014-03-31"
categories: [aero snap, Lubuntu, lubuntu, openbox, Tech]
comments: true
---

If you're using Lubuntu 13.10, then chances are that you can snap a window to
any of the screen edges (top, bottom, right or left) by pressing
`Windows+ArrowKey`. Or, if you're on a Mac, then `Command+ArrowKey`.

But these don't produce the exact same result as Windows 7 (or greater) does. The
`Windows+Right` and `Windows+Left` keys align the window to the right or left edge
but the window does not occupy the full height of the screen.
The `Windows+Up` and `Windows+Down` keys only take up half the screen.

These settings are set in the file `~/.config/openbox/lubuntu-rc.xml`. This is the relevant
snippet of the section:
```xml
<!-- Keybindings for window tiling -->
    <keybind key="W-Left">        # HalfLeftScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>0</x><y>0</y><height>97%</height><width>50%</width></action>
    </keybind>
    <keybind key="W-Right">        # HalfRightScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>-0</x><y>0</y><height>97%</height><width>50%</width></action>
    </keybind>
    <keybind key="W-Up">        # HalfUpperScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>0</x><y>0</y><width>100%</width><height>50%</height></action>
    </keybind>
    <keybind key="W-Down">        # HalfLowerScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>0</x><y>-0</y><width>100%</width><height>50%</height></action>
    </keybind>
```

Edit the file using your favorite text editor.

Change the number between `<height>x%</height>` to `100%`. But, that's not all.
The `Windows+Up` key will fill the entire screen but still keep the window
<em>unmaximized</em>, requiring you to click the maximize button.
This annoys me. To change this you can replace `"UnmaximizeFull"` with `"Maximize"` in
the `W-Up` section. And, you're done.

Now, save the file. The corresponding section of the edited file looks like this.
```xml
<!-- Keybindings for window tiling -->
    <keybind key="W-Left">        # HalfLeftScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>0</x><y>0</y><height>100%</height><width>50%</width></action>
    </keybind>
    <keybind key="W-Right">        # HalfRightScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>-0</x><y>0</y><height>100%</height><width>50%</width></action>
    </keybind>
    <keybind key="W-Up">        # HalfUpperScreen
      <action name="Maximize"/>
      <action name="MoveResizeTo"><x>0</x><y>0</y><width>100%</width><height>100%</height></action>
    </keybind>
    <keybind key="W-Down">        # HalfLowerScreen
      <action name="UnmaximizeFull"/>
      <action name="MoveResizeTo"><x>0</x><y>-0</y><width>100%</width><height>50%</height></action>
    </keybind>
```

Then, restart `openbox` by typing this in the terminal

```bash
openbox --restart
```
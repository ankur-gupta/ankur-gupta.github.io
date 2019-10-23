---
layout: post
identifier: search-and-destory-files
title: "Search & destroy files"
date: 2019-10-01
comments: true
---
It is a common task to recursively seek and destroy all `.pyc` files within current
directory. The following command does that for us in a `bash` shell.

```bash
find . -name "*.pyc" -type f -exec rm "{}" \;
```

Simply typing `find . -name "*.pyc" -type f` would list all files (and not directories)
whose name matches the glob `*.pyc` within the directory `.` (current directory).
`find` also accepts `-exec` option in both Linux and MacOS (though `-delete`
option [may not](https://unix.stackexchange.com/questions/167823/find-exec-rm-vs-delete)
always be available). See other variants
[here](https://unix.stackexchange.com/questions/167823/find-exec-rm-vs-delete).
The above command **works even for filenames or paths that have spaces** in them.
In contrast, using `find . -name "*.pyc" -type f | xargs rm` is risky when filenames or paths
contain spaces. Here is a demo:

```bash
# In a bash shell
$ find . -name "*.pyc" -type f
./hello world.pyc
./p q r/this has spaces.pyc
./p q r/d.pyc
./abc.pyc
./a/pqr.pyc

$ find . -name "*.pyc" -type f -exec rm "{}" \;
$ find . -name "*.pyc" -type f
# Nothing anymore!
```



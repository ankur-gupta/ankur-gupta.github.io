---
layout: post
title: "Install xml2 R package on MacOS"
date: 2016-04-08
categories: [R, libxml2, install, mac, xml2]
comments: true
---

Installing [`xml2`](https://github.com/hadley/xml2) `R` package often fails due to missing or incompatible library issues. In this post, I describe why this problem occurs and provide two solutions to solve this problem.

## What goes wrong?

`xml2` `R` package depends on `libxml2`. When you install `xml2` using `install.packages` and default options, `xml2` package queries your system to determine where `libxml2` headers and library files are located (more on this later).
The error occurs when `libxml2` is not found or a wrong version of `libxml2`.
This is how the error looks.

```r
> install.packages("xml2")
...
* installing *source* package ‘xml2’ ...
** package ‘xml2’ successfully unpacked and MD5 sums checked
Found pkg-config cflags and libs!
Using PKG_CFLAGS=-I/Users/<username>/anaconda/include/libxml2
Using PKG_LIBS=-L/Users/<username>/anaconda/lib -lxml2 -lz -liconv -lm
** libs
clang++ -I/usr/local/Cellar/r/3.2.2_1/R.framework/Resources/include -DNDEBUG -I/usr/local/include -I/Users/<username>/anaconda/include/libxml2 -I/usr/local/opt/gettext/include -I/usr/local/opt/readline/include -I/usr/local/opt/openssl/include -I/usr/local/include -I"/usr/local/lib/R/3.2/site-library/Rcpp/include" -I"/usr/local/lib/R/3.2/site-library/BH/include" -I/usr/local/include   -fPIC  -g -O2  -c RcppExports.cpp -o RcppExports.o
...
installing to /usr/local/lib/R/3.2/site-library/xml2/libs
...
Error in dyn.load(file, DLLpath = DLLpath, ...) :
  unable to load shared object '/usr/local/lib/R/3.2/site-library/xml2/libs/xml2.so':
  dlopen(/usr/local/lib/R/3.2/site-library/xml2/libs/xml2.so, 6): Library not loaded: libxml2.2.dylib
  Referenced from: /usr/local/lib/R/3.2/site-library/xml2/libs/xml2.so
  Reason: image not found
Error: loading failed
Execution halted
```

## What happens behind the scenes?

`xml2` package contains a bash script called `configure` which determines the location of `libxml2`. You can take a look at this [configure](https://github.com/hadley/xml2/blob/master/configure) on GitHub. This script first attempts to use `xml2-config` to determine the location. On my system, `xml2-config` is installed by `anaconda`.

```bash
$ which xml2-config
/Users/<username>/anaconda/bin/xml2-config
```

On my system, `xml2-config` tool points to the `libxml2` installed by `anaconda`, which happens to be incompatible with `R` `xml2` package.

```bash
$ xml2-config --libs
-L/Users/<username>/anaconda/lib -lxml2 -lz -liconv -lm
$ xml2-config --cflags
-I/Users/<username>/anaconda/include/libxml2
```

If `xml2-config` is not available, then `configure` script checks for `libxml2` using the `pkg-config` tool.

```bash
$ which pkg-config
/usr/local/bin/pkg-config
```

Again, in my case, this `pkg-config` tool points to the MacOS system `libxml2` headers and libraries. These may or may not be compatible with `R` `xml2` package. In my case, these happened to be compatible.

```bash
$ pkg-config --cflags libxml-2.0
-I/usr/include/libxml2
$ pkg-config --libs libxml-2.0
-lxml2
```

We need to make sure that the `/usr/include/libxml2` location actually exists.
If this location does not exist, then first solve this problem as described
in the footnotes[^1].

Overall, the solution is based on the simple principle

_Use the correct location of `libxml2` headers and libraries._

There are various ways to solve this problem. I describe two ways in this post.

## Possible solutions

For both solutions, we will need to download the [source code](https://cran.r-project.org/web/packages/xml2/index.html) of `xml2` package from CRAN and install the package from source. Extract the original source code file and navigate to the extracted `xml2` folder.

### Solution 1
We will modify the `configure` script in the source code. The modification will cause `configure` script to use the `libxml2` location
provided by `pkg-config`. This can be done by commenting out the following relevant lines in the `configure` file

```bash
# Use xml2-config if available
if [ $(command -v xml2-config) ]; then
  PKGCONFIG_CFLAGS=$(xml2-config --cflags)
  PKGCONFIG_LIBS=$(xml2-config --libs)
elif [ $(command -v pkg-config) ]; then
  PKGCONFIG_CFLAGS=$(pkg-config --cflags $PKG_CONFIG_NAME)
  PKGCONFIG_LIBS=$(pkg-config --libs $PKG_CONFIG_NAME)
fi
```

and replacing them by these lines

```bash
if [ $(command -v pkg-config) ]; then
  PKGCONFIG_CFLAGS=$(pkg-config --cflags $PKG_CONFIG_NAME)
  PKGCONFIG_LIBS=$(pkg-config --libs $PKG_CONFIG_NAME)
fi
```

I recommend reading the code and then made the changes if you understand them.

Now that we have modified the source code of the package, we need to re-build a new `.tar.gz` file.

```bash
# Navigate to inside the xml2 folder
~/Downloads/xml2 $ R CMD build .
# This will create a xmlx_x.x.x.tar.gz file in the current folder
```

and then install this package from from source

```r
# Start R in the folder that contains the xmlx_x.x.x.tar.gz file
> install.packages("xml2_0.1.2.tar.gz", repos=NULL, type="source")
```

If the location of `libxml2` as specified by `pkg-config` is correct, then this solution should correctly install `xml2` package.

## Solution 2
If you do not want to modify the `configure` script and you know the correct location of `libxml2` header and libraries, then you can simply perform
a custom install of `xml2` package by using the following command
(as specified by the `configure` script)

```bash
# Navigate to inside the xml2 folder
R CMD INSTALL --configure-vars='INCLUDE_DIR=/usr/include LIB_DIR=/usr/lib' .
```

The above locations for `INCLUDE_DIR` and `LIB_DIR` correspond to the default MacOS locations but you may need to modify these. You can find out these locations using the tips provided in footnotes[^2]. This should successfully install `xml2` package. This form of solution works correctly for other packages as well, albeit with some modifications[^3].

### Footnotes

[^1]: Missing `/usr/include` on MacOS

    Sometimes, `/usr/include` does not exist on MacOS because Xcode did not install correctly, as mentioned in this [Stack Overflow post](http://stackoverflow.com/questions/27328049/missing-usr-include-after-yosemite-and-xcode-install). The solution is simple, just run this command and follow instructions.

    ```bash
    xcode-select --install
    ```


[^2]: `libxml2` metadata file

    The `configure` script requires `libxml-2.0.pc` file at the location where `libxml2` is installed. You can look for all `libxml-2.0.pc` files on your system

    ```bash
    $ locate libxml-2.0.pc
    /Users/<username>/anaconda/lib/pkgconfig/libxml-2.0.pc
    /Users/<username>/anaconda/pkgs/libxml2-2.9.0-1/lib/pkgconfig/libxml-2.0.pc
    /opt/vagrant/embedded/lib/pkgconfig/libxml-2.0.pc
    /usr/local/Library/ENV/pkgconfig/10.10/libxml-2.0.pc
    /usr/local/Library/ENV/pkgconfig/10.11/libxml-2.0.pc
    ...
    ```

    `pkg-config` looks at these `.pc` metadata files to retrieve information,
    as mentioned in the manual pages of `pkg-config`

    ```bash
    $ man pkg-config
    ...
    pkg-config retrieves information about packages from special metadata files.
    These files are named after the package, and has a .pc extension.
    ...
    ```

    These `libxml-2.0.pc` files contain location of `libxml2` headers and libraries. This information is usually very helpful.

    ```bash
    $ more /usr/local/Library/ENV/pkgconfig/10.11/libxml-2.0.pc
    prefix=/usr
    exec_prefix=${prefix}
    libdir=${exec_prefix}/lib
    includedir=${prefix}/include
    modules=1

    Name: libXML
    Version: 2.9.2
    Description: libXML library version2.
    Requires:
    Libs: -L${libdir} -lxml2
    Libs.private: -lpthread -lz  -lm
    Cflags: -I${includedir}/libxml2
    ```
[^3]: Installing [`git2r`](https://cran.r-project.org/web/packages/git2r/index.html)

    A similar error somtimes occurs while installing `git2r`. On my system, the error was due to incorrect location of
    `zlib` library. Solution 2 works but the exact command is slightly different:

    ```bash
    # Navigate to inside the git2r source folder
    R CMD INSTALL --configure-args='--with-zlib-include=/usr/include --with-zlib-lib=/usr/lib' .
    ```

  The exact options required by `configure` may be seen by running `./configure --help` within the source directory
  of the package. Note that if you have run `configure` within the source folder using the incorect options, then
  you might see `R CMD INSTALL` fail even with the correct options. This problem can be solved by re-running
  `configure` using the correct options or getting a fresh copy of the source.



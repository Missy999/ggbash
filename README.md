<!-- README.md is generated from README.Rmd. Please edit that file -->
ggbash: A Faster Way to Write ggplot2
=====================================

[![Travis-CI Build Status](https://travis-ci.org/caprice-j/ggbash.svg?branch=master)](https://travis-ci.org/caprice-j/ggbash) [![Build status](https://ci.appveyor.com/api/projects/status/vfia7i1hfowhpqhs?svg=true)](https://ci.appveyor.com/project/caprice-j/ggbash) [![codecov](https://codecov.io/gh/caprice-j/ggbash/branch/master/graph/badge.svg)](https://codecov.io/gh/caprice-j/ggbash) <!-- [![Coverage Status](https://coveralls.io/repos/github/caprice-j/ggbash/badge.svg)](https://coveralls.io/github/caprice-j/ggbash) --> [![Issue Count](https://codeclimate.com/github/caprice-j/ggbash/badges/issue_count.svg)](https://codeclimate.com/github/caprice-j/ggbash/issues)

ggbash provides a bash-like REPL environment for [ggplot2](https://github.com/tidyverse/ggplot2).

Installation
------------

``` r
devtools::install_github("caprice-j/ggbash")
```

Usage
-----

``` bash
library(ggbash)
ggbash(iris) # start a ggbash session (using iris as main dataset)
```

``` bash
user@host currentDir (iris) $ p Sepal.W Sepal.L c=Sp si=Petal.W
```

``` r
executed:
    ggplot(iris) +
    geom_point(aes(Sepal.Width,
                   Sepal.Length,
                   color=Species,
                   size=Petal.Width))
```

![](README_files/figure-markdown_github/unnamed-chunk-6-1.png)

Or if you just need one figure,

``` r
executed <- drawgg(iris, 'p Sepal.W Sepal.L c=Sp siz=Petal.W')
copy_to_clipboard(executed$cmd)
# copied: ggplot(iris) + geom_point(aes(x=Sepal.Width, y=Sepal.Length, colour=Species, size=Petal.Width))
```

Features
--------

### 1. Column Index Match

``` bash
# 'list' displays column indices
user@host currentDir (iris) $ list

    1: Sepal.L  (ength)
    2: Sepal.W  (idth)
    3: Petal.L  (ength)
    4: Petal.W  (idth)      
    5: Spec     (ies)

# the same as the above 'p Sepal.W Sepal.L c=Sp si=Petal.W'
user@host currentDir (iris) $ p 2 1 c=5 si=4

# you can mix both notations
user@host currentDir (iris) $ p 2 1 c=5 si=Petal.W
```

Column Index Match is perhaps the fastest way to build a ggplot object. In the above case, while the normal ggplot2 notation (`ggplot(iris) + geom_point(aes(x=Sepal.Width, y=Sepal.Length, colour=Species, size=Petal.Width))`) contains 90 characters (spaces not counted), `p 2 1 c=5 si=4` is just **10 characters -- more than 80% keystroke reduction**.

With more elaborated plots, the differences become much larger.

### 2. Partial Prefix Match

In the first example (`p Sepal.W Sepal.L c=Sp si=Petal.W`), ggbash performs partial matches seven times.

-   **geom names**
    -   `p` matches `geom_point`
        -   Note: the common prefix geom\_ is removed beforehand.
-   **column names**
    -   `Sepal.W` matches `iris$Sepal.Width`.

    -   `Sepal.L` matches `iris$Sepal.Length`.

    -   `Sp` matches `iris$Species`.

    -   `Petal.W` matches `iris$Petal.Width`.

-   **aesthetics names**
    -   `c` matches `color`, which is the aesthetic of geom\_point.
    -   `si` matches `size` ('s' cannot identify which one of `shape`, `size`, and `stroke`).

Note: Approximate String Match (e.g. identifying `size` by `sz`)is not supported in the current version.

### 3. Predefined Precedence

Even if unique identification is not possible, `ggbash` tries to guess what is specified instead of bluntly returning an error, hoping to achieve least expected keystrokes.

For example, in the input `p Sepal.W Sepal.L c=Sp s=Petal.W`, `p` ambiguously matches four different geoms, `geom_point`, `geom_path`, `geom_polygon`, and `geom_pointrange`.
`ggbash` determines which geom to use by the above predefined order of precedence (the first geom, `point`, is selected in this example).

Similarly, `s` matches three aesthetics, `size`, `shape`, and `stroke`, preferred by this order. Thus, `s` is interpreted as the `size` aesthetic.

While it's possible to define your own precedence order through `define_constant_list()`, adding one or two characters might be faster in most cases.

### 4. Piping to copy/save results

``` r
    user@host currentDir (iris) $ cd imageDir

    user@host currentDir (iris) $ ls
    /Users/myname/currentDir/imageDir
    
    user@host imageDir (iris) $ p 2 1 c=5 si=4 | png big
    saved as 'iris-Sepal.W-Sepal.L-Sp.png' (big: 1960 x 1440)
    
    user@host imageDir (iris) $ for i in 2:5 p 1 i | pdf 'iris-for'
    saved as 'iris-for.pdf' (default: 960 x 960)
    
    user@host imageDir (iris) $ p 1 2 col=Sp siz=4 | copy
    copied to clipboard:
    ggplot(iris) + geom_point(aes(x=Sepal.Length,
                                  y=Sepal.Width,
                                  colour=Species,
                                  size=Petal.Width))
```

### 5. ggplot ID

TODO write me
=============

iris-p-Sepal.W-Sepal.L-col=Sp-sz=Petal.W.png \# TODO how can we encode scales/facets/themes differences? Scales are ... Facets are ... for is by pdf extension and i values Themes are abstracted away and not encoded in file names.

Goals
-----

ggbash has two main goals:

1.  *Better EDA experience:* Provide blazingly fast way to do Exploratory Data Analysis.

    -   less typing by Column Index Match, Partial Prefix Match, and Predefined Precedence.

    -   casualy save plots with auto-generated unique file names

2.  *Better tweaking experience:* Make it less stressful to finalize your plots.

    -   adjust colors or lineweights

    -   rotate axis labels

    -   decide tick label intervals and limits

    -   generate line-wrapped titles or legends

Learning ggbash
---------------

`ggbash` follows ggplot2 notations as much as possible for reducing learning costs of current ggplot2 users.

Therefore, these [document](http://docs.ggplot2.org/current/) and [book](https://github.com/hadley/ggplot2-book) are good ways to get the hang of ggplot2.

The [vignette](https://github.com/caprice-j/ggbash/blob/master/vignettes/Introduction-to-ggbash.Rmd) of ggbash is still a draft.

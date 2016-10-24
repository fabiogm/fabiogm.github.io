---
layout: post
title:  "Installing xgboost with OpenMP support on Mac OSX"
date:   2016-10-24 01:57:41 -0300
categories: programming data-science
---

After losing a few hours (and my temper in the process) I thought I would share the steps to compile and install xgboost on a Mac OSX with
OpenMP (for multi-threading) support. Official docs weren't terribly helpful.

This is going to be graceful.

First, download the code

```
git clone https://github.com/dmlc/xgboost/
```

Now get a compiler with OpenMP support. Installing a clang version newer than the default version that ships with Mac OSX
El Capitan turned out to be a bit of a hassle, so I went ahead with GCC instead. The easiest way I could find (it is in the official 
xgboost documentation) is using brew

```
brew install gcc --without-multilib
```

It might take a while, it took about an hour on a MacBook Pro 2.6 GHz Intel Core i5 with 8GB of RAM. Don't use `brew install --with-clang llvm`.
IT'S NOT GOING TO WORK. Or maybe it will, I don't know, I tried that before discovering how to point R to the right compiler.

Assuming you now have a compiler with OpenMP support comes the trickiest part: point R to the right compiler.

I tried [this](http://stackoverflow.com/questions/23414448/r-makevars-file-to-overwrite-r-cmds-default-g-options) solution, it didn't work.

Also tried a couple more approaches, to no avail. I also didn't feel like fiddling with symlinks and brew links. 

That's how a discovered this one weird trick to get ~~six pack abs~~ R compiling with OpenMP support.

It's not pretty, but you can edit default R `Makevars` file. This is a makefile that holds several flags, including compilers and OpenMP support.
You can find the location of this file on your machine using this command

```
file.path(R.home("etc"), "Makeconf")
```

Mine is at `/Library/Frameworks/R.framework/Resources/etc/Makeconf`.

Open the file and change all references to `clang` to point to `gcc-6` instead. At the time of this writing GCC 6 is the most recent version, yours 
might be different.

After this type inside R.

```
install.packages('devtools')
library(devtools)
install('xgboost/R-package')
```

And _et voilà_, you should be all set.

If you feel like trying a less hacky solution, [this thread](https://github.com/dmlc/xgboost/issues/276) might help.

---
layout: post
title:  "Solving RStudio Missing Latex Packages Issue"
date:   2017-01-21 13:20:00 -0300
categories: other
---

This post is more of a note to self for next time I encounter the same problem. When trying to generate a PDF from an RMarkdown file using
 Rstudio I got the following error

```
! LaTeX Error: File `framed.sty' not found.
```

Interestingly enough this error started to happen after I updated RStudio. Before the update I was able to generate PDF files without
 problems. However, I do remember having to install Latex packages in the past before generating any PDF files. How did installed packages
 disappear from my system after an update? Mystery. Anyway, if you use MacTeX distribution the solution is simple: you can use the
 package manager that comes bundled with MacTeX to install the missing package:

```
sudo tlmgr update --self
sudo tlmgr install framed
```

Problem solved? Not yet. After installing `framed` it turns out I had one more package missing:

```
! LaTeX Error: File `titling.sty' not found.
``` 

The solution, once again, is to install the missing package:

```
sudo tlmgr install titling
```

That should solve this common problem. In case you have more errors like those, try to identify the name of the package that contains
 the missing file RStudio (more specifically pandoc) is complaining about and install that package.


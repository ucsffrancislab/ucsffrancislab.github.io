
#	Rmarkdown



For some reason, the filename extension either needs to be `.rmd` or can't be `.txt`. 

I first tried this downloaded file which arrived as `.txt` and it didn't execute any of the R code.


```
R -e "rmarkdown::render('example-r-markdown.rmd',output_file='example-r-markdown.rmd.html')"

open example-r-markdown.rmd.html 
```



I don't think that it needs to be broken up into smaller blocks unless you want to.
I think that rmarkdown will break it up if anything is output.
Of course, if you need to add your own text, you will need to do it manually.



##	Self executable ...


Start your `myfile.Rmd` with ...

```
#!/usr/bin/env Rscript

args <- commandArgs()
fname <- normalizePath(sub("--file=", "", args[grepl("--file=", args)]))
thisfile <- readLines(fname)
newfname <- paste0(tempdir(), "/", basename(fname))
writeLines(thisfile[-1:-which(thisfile == "q(\"no\")")], newfname)

args = commandArgs(trailingOnly=TRUE)
#output_file = paste(basename(fname),basename(args[2]),"html", sep=".")
output_file = paste(basename(fname),"html", sep=".")
print(output_file)

rmarkdown::render(newfname, output_dir = dirname(fname), output_file = output_file )
q("no")
```

Anything after the `q("no")` line will be part of the actual rmarkdown.

This also allows for the use of positional arguments. 
In this case, I use one in the initial script and later on I ...

```{r}
meta=args[1]
ifile=args[2]
```


I still don't get the difference between `=` and `<-` in R. I've used them interchangeably with no obvious issues.



##	Rmarkdown example


Complete example of self executing script that takes command line arguments,
that could be used to control the output file,
includes a table of contents, 
2 different tab sets
and could use python.




````
#!/usr/bin/env Rscript

args <- commandArgs()
fname <- normalizePath(sub("--file=", "", args[grepl("--file=", args)]))
thisfile <- readLines(fname)
newfname <- paste0(tempdir(), "/", basename(fname))
writeLines(thisfile[-1:-which(thisfile == "q(\"no\")")], newfname)

args = commandArgs(trailingOnly=TRUE)
#output_file = paste(basename(fname),basename(args[2]),"html", sep=".")
output_file = paste(basename(fname),"html", sep=".")
print(output_file)

rmarkdown::render(newfname, output_dir = dirname(fname), output_file = output_file )
q("no")


---
title: RMD TEST
author: Jake
date: 20221122
output: 
  html_document:
    toc: true # table of content true
    toc_depth: 3  # upto three depths of headings (specified by #, ## and ###)
    number_sections: true  ## if you want number sections at each table header
    theme: united  # many options for theme, this one is my favorite.
    highlight: tango
---


#	Description

This Rmd is self executed.

Take command line arguments.

Build a Table of Contents.

And demonstrates tab sets.


# Load libraries and read data {.tabset .tabset-fade .tabset-pills}

<!--

THAT MUST BE FIRST TO BE USEFUL!

-->


```{r "Figure Settings", include = FALSE}
knitr::opts_chunk$set(fig.width=12, fig.height=8) 
```

USE THIS TO SET ALL DEFAULTS

```{r defaults, include=FALSE}
    knitr::opts_chunk$set(
error=TRUE, # my new favorite, will let the script run and create html so you could debug
      comment = '', # Remove comments from the output
      fig.width = 6, # Set default plot width
      fig.height = 6, # Set default plot height
      echo = TRUE # Echo code by default
    )   
```


# Tabset example {.tabset .tabset-fade .tabset-pills}

##	One
```{r setup}
library('reticulate')
```

##	Two
```{r python}
use_python('/opt/local/bin/python')
```

##	Three
```{r args}
args
```

##	Four
```{python pyargs}
r.args
```


# Another different style {.tabset .tabset-fade}

##	One

Content for One

##	Two

Content for Two

##	Three

Content for Three


````



Note that using the self-executing script will require that file parameters include a full path
as it will be run in the temp dir and a file in the current directory won't be in the current directory
when executed.




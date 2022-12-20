
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

output_file = paste(basename(fname),basename(args[2]),"html", sep=".")
print(output_file)




rmarkdown::render(newfname, output_dir = dirname(fname), output_file = output_file )
q("no")
```

Anything after the `q("no")` line will be part of the actual rmarkdown.

This also allows for the use of positional arguments. 
In this case, I use one in the initial script and later on I ...

```{r}
meta=args[0]
ifile=args[1]
```


###	Python




```{r setup}
library('reticulate')
```

```{r python}
use_python('/opt/local/bin/python')
```

Access the args from R

```{r args}
args
```

Or from python ...

```{python pyargs}
r.args
```







I still don't get the difference between `=` and `<-` in R. I've used them interchangeably with no obvious issues.



##	Rmarkdown example

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
```{}


# Tabset example {.tabset .tabset-fade .tabset-pills}

##	One
```{r setup}
library('reticulate')
```{}

##	Two
\```{r python}
use_python('/opt/local/bin/python')
\```

##	Three
\```{r args}
args
\```

##	Four
\```{python pyargs}
r.args
\```


# Another different style {.tabset .tabset-fade}

##	One

Content for One

##	Two

Content for Two

##	Three

Content for Three


```



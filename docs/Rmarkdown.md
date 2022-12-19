
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


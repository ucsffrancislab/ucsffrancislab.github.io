
#	Rmarkdown



For some reason, the filename extension either needs to be `.rmd` or can't be `.txt`. 

I first tried this downloaded file which arrived as `.txt` and it didn't execute any of the R code.


```
R -e "rmarkdown::render('example-r-markdown.rmd',output_file='example-r-markdown.rmd.html')"

open example-r-markdown.rmd.html 
```



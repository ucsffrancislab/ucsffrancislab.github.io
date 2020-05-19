# UCSF Francis Lab


https://ucsffrancislab.github.io




```
ls -1 docs/* | awk -F/ '{split($2,a,".");split($0,b,".");print "* ["a[1]"]("b[1]")"}' >> index.md
```


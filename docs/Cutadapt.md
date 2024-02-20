
#	Cutadapt

##	Install

```
python3 -m pip install --user --upgrade cutadapt
```



## Usage


A simple, identical pair of reads with a polyA in the middle


```
cat cutadapt.R1.fa
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT

cat cutadapt.R2.fa
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
```


It is my understanding the R1's are right trimmed (-a)
and R2's are left trimmed (-G)



```
cutadapt --times 1 -a A{10} -o out.R1.fa -p out.R2.fa cutadapt.R1.fa cutadapt.R2.fa

cat out.R?.fa
>1
GTGTAGATCTCGGTGGTCGC
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
```

```
cutadapt --times 1 -G A{10} -o out.R1.fa -p out.R2.fa cutadapt.R1.fa cutadapt.R2.fa

cat out.R?.fa
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
>1
AAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
```

Both -a and -G choose the first from the left and trim in their direction. 
They ARE NOT greedy by trimming the largest. 
This seems odd and perhaps I'm doing something wrong.

In order to force the issue and make -G greedy you need to either make the poly string arbitrarilty longer or increase the --times


```
cutadapt --times 2 -G A{10} -o out.R1.fa -p out.R2.fa cutadapt.R1.fa cutadapt.R2.fa

cat out.R?.fa
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
>1
AAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
```

```
cutadapt --times 3 -G A{10} -o out.R1.fa -p out.R2.fa cutadapt.R1.fa cutadapt.R2.fa

cat out.R?.fa
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
>1
AAAAAAGTCTCGGTGGTCGCCGTATCATT
```

```
cutadapt --times 4 -G A{10} -o out.R1.fa -p out.R2.fa cutadapt.R1.fa cutadapt.R2.fa

cat out.R?.fa
>1
GTGTAGATCTCGGTGGTCGCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGTCTCGGTGGTCGCCGTATCATT
>1
GTCTCGGTGGTCGCCGTATCATT
```








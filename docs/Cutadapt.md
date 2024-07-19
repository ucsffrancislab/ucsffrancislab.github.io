
#	Cutadapt

##	Install

```
python3 -m pip install --user --upgrade cutadapt
```




##	Prep

Trying to verbalise all this.


What is the orientation of a proper pair of reads?


If we extract a proper pair aligned reads we get ...

```
samtools view -f 2 1_10.format.umi.trim.Aligned.sortedByCoord.out.umi_tag.fixmate.deduped.bam | head -2

VH00749:75:AACVVCTM5:1:1105:57068:17225	99	chr1	10001	255	31S101M	=	10001	103	GGCACCCTAACCCTAACCCTAAACCTAACCCTAACCCTAACCCTAACCCAAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACC	CCCCCCCCCCCCCCCCCCCCCC-CCCCCCCCCCCCCCCCCCCCCCCCCC;C-CCCC;CCCCCCCCCCCCC-CCC;CC-C--CCCCCC-CCCCCCCCCCCCC;CC-CC;-CCCCC;--CCCCCCCCCCC;CCC	MC:Z:20S103M	RG:Z:1_10	NH:i:1	HI:i:1	nM:i:2	MQ:i:255	AS:i:198	RX:Z:GTAAGTTGGTTGGTTAGG
VH00749:75:AACVVCTM5:1:1105:57068:17225	147	chr1	10001	255	20S103M	=	10001	-103	CCTATCCCTAACCCTAACCCTAACCCTAACCCTAACACTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCT	CCC;-CCCCCCCC-CCCCCC-C;;-;CCCC-CCCC--;-C;CCCCC-C--CCC-C;-CCC-;-C-CCCC-C;CC;CCC-CCC;CCCCCCCCC;CCCCCCC;CCCCCCCCCCCCCCCCC-CCC-	MC:Z:31S101M	RG:Z:1_10	NH:i:1	HI:i:1	nM:i:2	MQ:i:255	AS:i:198	RX:Z:GTAAGTTGGTTGGTTAGG
```

The first read is forward, the second is reverse complemented.
```
samtools flags 99
0x63	99	PAIRED,PROPER_PAIR,MREVERSE,READ1

samtools flags 147
0x93	147	PAIRED,PROPER_PAIR,REVERSE,READ2
```

We see here the reads in the bam and the fastq, R1 are the same, and R2 are reverse complemented.
```
samtools view -f 2 1_10.format.umi.trim.Aligned.sortedByCoord.out.umi_tag.fixmate.deduped.bam | head -2 | awk '{print $10}'
GGCACCCTAACCCTAACCCTAAACCTAACCCTAACCCTAACCCTAACCCAAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACC
CCTATCCCTAACCCTAACCCTAACCCTAACCCTAACACTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCT

zcat 1_10.format.umi.trim.Aligned.sortedByCoord.out.umi_tag.fixmate.deduped.R1.fastq.gz | head -2
@VH00749:75:AACVVCTM5:1:1105:57068:17225/1
GGCACCCTAACCCTAACCCTAAACCTAACCCTAACCCTAACCCTAACCCAAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAACC

zcat 1_10.format.umi.trim.Aligned.sortedByCoord.out.umi_tag.fixmate.deduped.R2.fastq.gz | head -2
@VH00749:75:AACVVCTM5:1:1105:57068:17225/2
AGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGTTAGTGTTAGGGTTAGGGTTAGGGTTAGGGTTAGGGATAGG

```


```
PREADAPTER - SEQUENCE - POLYA - POSTADAPTER
```


```
R1 -    PREADAPTER - SEQUENCE - POLYA   -    POSTADAPTER

R2 - RCPOSTADAPTER -  RCPOLYA - RCSEQUENCE - RCPREADAPTER
```


I think that the sequencer removes the PREADAPTER during processing. Think that call it an index?


So trim with `-a POLYA` and `-G RCPOLYA`


Both of these adapters trim the leftmost first so `-a` is greedy and `-G` is NOT greedy.`

Because `-G` is trimming from the left and not greedy, we need to increase `--times` 
```
--times 12
```

or make it longer 
```
-G T{20}
```

or include varying length of it
```
-G T{10} -G T{20} -G T{30} -G T{40}
```

Not sure which is best









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









##	Check Check Check


Fragments submitted to sequencer ???

```
P5 - I5 - R1 Primer - Sequence - poly A - R2 Primer (RC?) - I7 - P7

```


Sequencer returns what? They left trim at some point and return everything after?

R1 reads the main sequence left to right
R2 effectively reads the mate right to left

```
R1

R2

```



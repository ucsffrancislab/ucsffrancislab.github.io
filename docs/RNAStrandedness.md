
#	RNA Strandedness


After some stumbling around, I believe the following to be the case.

First runs of TEProF2 were done on Joe Costello's Spatial RNAseq data. 
This data turns out to have been prepared "strand specific". 
When running TEProF2, stringtie more specifically, not specify the strandedness or specifying the 
wrong strandedness results in empty data files after which the pipeline effectively fails. 
Specifying the correct strandedness results in success. 
Not quite sure why, but its all or nothing. Not some, most or half.

Strandedness can be determined using tools like RseQC, sadly post-alignment.
RseQC produces percentages of reads that are one strandedness or the other.
It is usually 90+% one way or the other, or it is nearly 50/50.

Not all data is strand specific and the strandedness needs to be specified on a read by read basis.
This is done using the XS tag in a bam file.
Given that RseQC isn't 100%, having the XS tag and not specifying strandedness to TEProF2 seems to be the most promising strategy.

STAR can add the XS tag if the options `--outSAMstrandField intronMotif --outSAMattributes Standard XS` are given.
```
“The XS flag is assigned only for spliced reads based on the intron motif of the junction.
For an unstranded library, the read may be sequenced from the 1st cDNA strand (opposite to RNA), 
but the intron motif allows to determine the actual RNA strand.”
```
In addition, STAR's github repository also includes an awk script `tagXSstrandedData.awk` which can be used to add an XS tag.
This script will add an XS tag to EVERY read. 
* READ1,REVERSE : +
* READ2,REVERSE : -
* READ1,MREVERSE : -
* READ2,MREVERSE : +

Not sure of the impact of adding it during alignment versus adding it after.

I took a single sample, TCGA-02-0047-01A-01R-1849-01+1, and processed it a number of different ways.

* aligning to a an old reference built without a GTF
* aligning to a an old reference built without a GTF then add XS with awk script
* aligning to a reference built without a GTF
* aligning to a reference built without a GTF then add XS with awk script
* aligning to a reference built with a GTF
* aligning to a reference built with a GTF then add XS with awk script
* aligning to a reference built with a GTF and add XS during alignment

I am currently running TEProF2 on this collection of 7 bams files at the moment.
All bams without XS have produced empty intermediate files.


...






Our of curiousity, I also used the awk script to add the XS tag to an bam file that had an XS tag added during alignment.

```
samtools view -c gtf_xs_then_xs_again.Aligned.sortedByCoord.out.bam
132792040
```

The bam file contains 132,792,040 reads.
Initial alignment added XS tags to 34,029,104 reads, just over 25%.
After the awk script was run, reads that already had an XS tag from alignment, also included the additional one.
Sadly, roughly half are different.

```
samtools view gtf_xs_then_xs_again.Aligned.sortedByCoord.out.bam | grep -E -o "XS:A:.*XS:A:.*$" | sort | uniq -c
8782652 XS:A:-	XS:A:-
8772158 XS:A:-	XS:A:+
8229596 XS:A:+	XS:A:-
8244698 XS:A:+	XS:A:+
```

This seems to be a problem. Why are they different? Which is correct?

It appears that un-stranded RNAseq data needs to have the XS tag produced during alignment.

Stranded RNAseq data can either have the XS tag added using the awk script or by not including the XS tag and specifying the strand during other processing.





To test this, I'll add the XS tag using the awk script to a bam file that had the XS tag added during alignment from a stranded data set, the Costello Spatial RNAseq data set.


```
samtools view -c /francislab/data1/working/20200609_costello_RNAseq_spatial/20230424-STAR_hg38_strand/out/485v10.Aligned.sortedByCoord.out.bam
245740996

samtools view /francislab/data1/working/20200609_costello_RNAseq_spatial/20230424-STAR_hg38_strand/out/485v10.Aligned.sortedByCoord.out.bam | grep XS:A: | wc -l
98637018
```

just over 40% tagged with XS

```
samtools view -h /francislab/data1/working/20200609_costello_RNAseq_spatial/20230424-STAR_hg38_strand/out/485v10.Aligned.sortedByCoord.out.bam | awk -v strType=2 -f /francislab/data1/working/20200720-TCGA-GBMLGG-RNA_bam/20230628-STAR-TEProF2-test/tagXSstrandedData.awk | samtools view -h -o 485v10_then_XS.Aligned.sortedByCoord.out.bam -

samtools view 485v10_then_XS.Aligned.sortedByCoord.out.bam | grep -E -o "XS:A:.*XS:A:.*$" | sort | uniq -c

49290394 XS:A:-	XS:A:-
  214228 XS:A:-	XS:A:+
  245896 XS:A:+	XS:A:-
48886500 XS:A:+	XS:A:+
```
<0.5% mismatch


I'm not sure that I understand the purpose or advantage of creating a stranded library but knowing which we have is important.





Still waiting on the TEProF2 results of comparing the datasets





##	References


https://physiology.med.cornell.edu/faculty/skrabanek/lab/angsd/lecture_notes/STARmanual.pdf
```
For unstranded RNA-seq data, Cufflinks/Cuffdiff require spliced alignments with XS strand attribute,
which STAR will generate with --outSAMstrandField intronMotif option. As required, the XS
strand attribute will be generated for all alignments that contain splice junctions. The spliced
alignments that have undefined strand (i.e. containing only non-canonical unannotated junctions)
will be suppressed.
If you have stranded RNA-seq data, you do not need to use any specific STAR options. Instead,
you need to run Cufflinks with the library option --library-type options. For example, cufflinks
... --library-type fr-firststrand should be used for the “standard” dUTP protocol, including Illumina’s stranded Tru-Seq. This option has to be used only for Cufflinks runs and not for STAR
runs
```


https://github.com/alexdobin/STAR/issues/567
```
The XS flag is assigned only for spliced reads based on the intron motif of the junction.
For an unstranded library, the read may be sequenced from the 1st cDNA strand (opposite to RNA), but the intron motif allows to determine the actual RNA strand.
Cheers,
Alex
```

https://groups.google.com/g/rna-star/c/45Bgi504lzw
```
for unstranded libraries, you can use --outSAMstrandField intronMotif option.
For stranded libraries, you would need to add the XS flag to the BAM file after mapping.
You can use this script from STAR distribution as an example of how to do it:
extras/scripts/tagXSstrandedData.awk

Cheers
Alex
```


https://groups.google.com/g/rna-star/c/t-oT4wUFFow
```
the XS strand is only added to spliced alignments based on their intron motifs.
It does not take into account the strandedness information.
For stranded RNA-seq, Cufflinks can be run with the --library-type parmameter, while StringTie for some reason lacks this option, and requires XS to be supplied for each alignment. I have a simple script that can add XS flag to all alignments for stranded data:
```



https://www.biostars.org/p/62974/
```
If you have un-stranded RNA-seq data, and wish to run Cufflinks/Cuffdiff on STAR alignments, you will need to use --outSAMstrandField intronMotif option, which will generate the XS strand attribute for all alignments that contain splice junctions. The spliced alignments that have undefined strand (i.e. containing only non-canonical junctions) will be suppressed.

If you have stranded RNA-seq data, this option is not required – instead, you need to use the correct library option --library-type options for Cufflinks runs. For example, --library-type fr-firststrand should be used for the “standard” dUTP protocol.
```



















##	Determining strandedness



Use RseQC to infer strandedness needed by TEProF2 / Stringtie



https://chipster.csc.fi/manual/rseqc_infer_rnaseq_experiment.html


https://rseqc.sourceforge.net/




Download gene models (update on 12/14/2021)

https://sourceforge.net/projects/rseqc/files/BED/Human_Homo_sapiens/

https://sourceforge.net/projects/rseqc/files/BED/Human_Homo_sapiens/hg38_GENCODE_V42_Basic.bed.gz/download




I think that the last line makes it "stranded", "--fr" or "fr-firststrand". NOPE.




Running RseQC returns all like ...

Fraction of reads explained by "1+-,1-+,2++,2--": 0.9568

https://chipster.csc.fi/manual/library-type-summary.html

This suggests fr-firststrand

Stringtie
--rf	Assumes a stranded library fr-firststrand.



```
For non strand specific experiment the ratio of explanations should be close to 50:50

Fraction of reads failed to determine: 0.0000
Fraction of reads explained by "1++,1--,2+-,2-+": 0.5074
Fraction of reads explained by "1+-,1-+,2++,2--": 0.4926
For strand specific experiments there are two scenarios:

Reads in file 1 are always on the same strand as the gene: ( --fr / fr-secondstrand )

Fraction of reads failed to determine: 0.0072
Fraction of reads explained by "1++,1--,2+-,2-+": 0.9441
Fraction of reads explained by "1+-,1-+,2++,2--": 0.0487
 
Reads in file 2 are always on the same strand as the gene: ( --rf / fr-firststrand )

Fraction of reads failed to determine: 0.0011
Fraction of reads explained by "1++,1--,2+-,2-+": 0.0053
Fraction of reads explained by "1+-,1-+,2++,2--": 0.9936
```






```
mkdir strand_check
for bai in /raleighlab/data1/naomi/HKU_RNA_seq/Data_Analysis/FASTQ/Trimmed_Data/Arriba/*Aligned.sortedByCoord.out.bam.bai; do
bam=${bai%%.bai}
echo $bam
f=strand_check/$(basename ${bai} .bam.bai).strand_check.txt
if [ ! -f ${f} ] ; then infer_experiment.py -r /francislab/data1/refs/RseQC/hg38_GENCODE_V42_Basic.bed -i ${bam} > ${f}
fi
done
```

```
for f in strand_check/*.strand_check.txt ; do
awk -F: '($1=="Fraction of reads explained by \"1+-,1-+,2++,2--\""){if($2<0.1){print "--fr"}else if($2>0.9){print "--rf"}else{print "none"}}' ${f}
done | sort | uniq -c
```




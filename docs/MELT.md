
#	MELT

https://melt.igs.umaryland.edu



##	Problems

Putting this first since its kinda important.


###	Genotyping Failure

When analyzing MELT-SPLIT, sometimes the Genotyping step fails on some samples.


```
Performing MELT analysis...
Failed at genotyping of file FG-6689-01A-11D-1891...
Exiting!


-------------------JAVA STACK TRACE-------------------
java.lang.NullPointerException
        at MELT.utilities.MELTSequenceUtilities.qualityString(MELTSequenceUtilities.java:33)
        at MELT.MELTIllumina.confirmAndCompare.ConfirmMethods.decidePairs(ConfirmMethods.java:344)
        at MELT.MELTIllumina.confirmAndCompare.ConfirmMethods.call(ConfirmMethods.java:100)
        at MELT.MELTIllumina.confirmAndCompare.ConfirmMain.doWork(ConfirmMain.java:161)
        at MELT.MELTIllumina.runtimes.StepThree.RunStepThree(StepThree.java:17)
        at MELT.MELTImplement.main(MELTImplement.java:81)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.eclipse.jdt.internal.jarinjarloader.JarRsrcLoader.main(JarRsrcLoader.java:61)

MELT version: 2.2.2
-------------------END STACK TRACE!-------------------
```

It results in a geno file which is short and then I have to move the sample out of that directory so that I can create a VCF without it.

Results are the same when rerunning.



###	VCF REF column inaccurate

When analyzing MELT-SPLIT, roughly 50% of the REFs in the VCF and not the bases in the actual reference.
On many occasions it isn't even what is in the bam files.

When running MELT-SINGLE, it always seems to be the same as the reference. (Not true)

This makes it difficult to fabricate a prior list which has to be a valid VCF.

It also makes me not trust this tool.

Being that it is no longer maintained and supported, we can't even really discuss it.



```
grep 175906272 MELT-2.2.2-SINGLE-bwa-06-0125-10A-01D-1490.SVA.final_comp.vcf
chr3	175906272	.	a	<INS:ME:SVA>	.	ac0	TSD=null;ASSESS=1;INTERNAL=null,null;SVTYPE=SVA;SVLEN=394;MEINFO=SVA,921,1315,+;DIFF=0:n1-920,g938a,c945a,g947a,n954-955,t957c,c984a,g995a,c999t,g1005a,n1008-1316;LP=2;RP=4;RA=-1;PRIOR=false;SR=0	GT:GL:DP:AD	0/0:-0.05,-22.88,-338.1:38:0

grep 35176560 MELT-2.2.2-SINGLE-bwa-06-0125-10A-01D-1490.SVA.final_comp.vcf
chr5	35176560	.	t	<INS:ME:SVA>	.	ac0	TSD=null;ASSESS=1;INTERNAL=NM_000949,INTRONIC;SVTYPE=SVA;SVLEN=413;MEINFO=SVA,902,1315,-;DIFF=0.07:n1-901,a906g,g910a,g914t,g918a,c920t,g938a,c945a,g947a,n954-955,t957c,n995-1059,g1063a,g1067a,a1079g,g1082t,n1100,a1102g,t1124c,n1136,n1161-1316;LP=3;RP=2;RA=0.585;PRIOR=false;SR=0	GT:GL:DP:AD	0/0:-2.45,-34.92,-508.3:58:0

grep 4918078 MELT-2.2.2-SINGLE-bwa-06-0125-10A-01D-1490.SVA.final_comp.vcf
chr18	4918078	.	t	<INS:ME:SVA>	.	ac0	TSD=null;ASSESS=1;INTERNAL=null,null;SVTYPE=SVA;SVLEN=357;MEINFO=SVA,958,1315,+;DIFF=0:n1-957,g995a,c999t,g1005a,g1009t,g1010a,t1029c,n1047-1139,c1177a,g1183a,t1197a,a1213g,n1218-1316;LP=3;RP=2;RA=0.585;PRIOR=false;SR=0	GT:GL:DP:AD	0/0:-1.01,-28.3,-364.9:47:0

grep 37630672 MELT-2.2.2-SINGLE-bwa-06-0125-10A-01D-1490.SVA.final_comp.vcf
chr19	37630672	.	a	<INS:ME:SVA>	.	ac0	TSD=null;ASSESS=1;INTERNAL=NM_014898,TERMINATOR;SVTYPE=SVA;SVLEN=425;MEINFO=SVA,890,1315,+;DIFF=0.07:n1-889,a899c,t900a,t901a,a906g,g910a,g914t,g918a,c920t,g938a,c945a,g947a,n954-955,t957c,n995-1038,a1053g,c1056a,g1063a,g1067a,n1076,a1079g,n1108-1316;LP=1;RP=4;RA=-2;PRIOR=false;SR=0	GT:GL:DP:AD	0/0:-1.12,-19.27,-256:32:0

```

The reference values and the values in the bam files for these positions are the same (C, A, G, G) and not those.
No idea where the a,t,t,a values are coming from.
I've looked in the raw data and the bam.disc data. The disc data is just less.


```
samtools_bases_at_position.bash -c chr3 -p 175906272 -b 06-0125-10A-01D-1490.bam | sort | uniq -c
     50 C
      1 G
samtools_bases_at_position.bash -c chr5 -p 35176560 -b 06-0125-10A-01D-1490.bam | sort | uniq -c
     59 A
      1 T
samtools_bases_at_position.bash -c chr18 -p 4918078 -b 06-0125-10A-01D-1490.bam | sort | uniq -c
     45 G
samtools_bases_at_position.bash -c chr19 -p 37630672 -b 06-0125-10A-01D-1490.bam | sort | uniq -c
     36 G
```

```
samtools faidx /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/20180810/hg38.chrXYM_alts.fa chr3:175906272-175906272
>chr3:175906272-175906272
c

samtools faidx /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/20180810/hg38.chrXYM_alts.fa chr5:35176560-35176560
>chr5:35176560-35176560
a

samtools faidx /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/20180810/hg38.chrXYM_alts.fa chr18:4918078-4918078
>chr18:4918078-4918078
g

samtools faidx /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/20180810/hg38.chrXYM_alts.fa chr19:37630672-37630672
>chr19:37630672-37630672
g
```











##	CloudMELT

Cloud MELT is a modified version of MELT designed to be run on AWS.
It looks overcomplicated and uses CWL to control the workflow.

https://genome.cshlp.org/content/31/12/2225.long

https://github.com/Scott-Devine/CloudMELT


CloudMELT currently requires that the files be accessible via wget/curl on http/https.
This means that it would NOT work with data on Amazon S3 (which is odd given that this was designed for running on AWS).
Nor would it work directly with TCGA/TARGET data on the GDC Data Portal.
Both would require downloading locally first so would at minimum require a modification to their pipeline.

Not sure exactly what advantage CloudMELT provides over just running MELT.
It is not multithreaded so perhaps larger data sets would reveal something. 
I'm gonna try the 3 different versions that I have to see if the group processing steps (2 and 4 I think) are any faster in their version.





##	Trial run

Trial run of the standard version of MELT.


###	Example from manual

7.4.6. Example MELT-SPLIT Workflow

As a simulated example to demonstrate MELT-SPLIT based MEI discovery, suppose we have sequenced four human individuals (Person1.sorted.bam, Person2.sorted.bam, Person3.sorted.bam, Person4.sorted.bam) to approximately 30x coverage. We then used bwa-mem to align them to the Hg19 human reference sequence. We now want to know if they have any LINE1 non-reference Mobile Element Insertions. We first want to run Preprocess on all four samples to ensure MELT has the proper information to discover MEIs:

```
java -Xmx2G -jar MELT.jar Preprocess /path/to/Person1.sorted.bam

java -Xmx2G -jar MELT.jar Preprocess /path/to/Person2.sorted.bam

java -Xmx2G -jar MELT.jar Preprocess /path/to/Person3.sorted.bam

java -Xmx2G -jar MELT.jar Preprocess /path/to/Person4.sorted.bam
```

Next, we want to do initial MEI site discovery in all 4 genomes (only required options shown):

```
java -Xmx6G -jar MELT.jar IndivAnalysis \
    -bamfile /path/to/Person1.sorted.bam \
    -w ./LINE1DISCOVERY/ \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa;

java -Xmx6G -jar MELT.jar IndivAnalysis \
    -bamfile /path/to/Person2.sorted.bam \
    -w ./LINE1DISCOVERY/ \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa;

java -Xmx6G -jar MELT.jar IndivAnalysis \
    -bamfile /path/to/Person3.sorted.bam \
    -w ./LINE1DISCOVERY/ \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa;

java -Xmx6G -jar MELT.jar IndivAnalysis \
    -bamfile /path/to/Person4.sorted.bam \
    -w ./LINE1DISCOVERY/ \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa;
```

Next, we want to combine the initial discovery across these four genomes into a single piece of information to aid genotyping and filtering of false positive hits:

```
java -Xmx4G -jar MELT.jar GroupAnalysis \
    -discoverydir ./LINE1DISCOVERY/ \
    -w ./LINE1DISCOVERY/ \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa \
    -n /path/to/human_annotation.bed;
```

Next, we will genotype each of the samples using the merged MEI information file:

```
java -Xmx2G -jar MELT.jar Genotype \
    -bamfile /path/to/Person1.sorted.bam \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa \
    -w ./LINE1DISCOVERY/ \
    -p ./LINE1DISCOVERY/;

java -Xmx2G -jar MELT.jar Genotype \
    -bamfile /path/to/Person2.sorted.bam \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa \
    -w ./LINE1DISCOVERY/ \
    -p ./LINE1DISCOVERY/;

java -Xmx2G -jar MELT.jar Genotype \
    -bamfile /path/to/Person3.sorted.bam \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human_reference.fa \
    -w ./LINE1DISCOVERY/ \
    -p ./LINE1DISCOVERY/;

java -Xmx2G -jar MELT.jar Genotype \
    -bamfile /path/to/Person4.sorted.bam \
    -t /path/to/LINE_MELT.zip \
    -h /path/to/human\_reference.fa \
    -w ./LINE1DISCOVERY/ \
    -p ./LINE1DISCOVERY/;
```

Finally, we provide the genotyping directory from Genotype (as -genotypingdir) to MakeVCF to generate the final VCF file:

```
java -Xmx2G -jar MELT.jar MakeVCF \
    -genotypingdir ./LINE1DISCOVERY/ \
    -h /path/to/human_reference.fa \
    -t /path/to/LINE_MELT.zip \
    -w ./LINE1DISCOVERY/ \
    -p ./LINE1DISCOVERY/LINE1.pre_geno.tsv;
```

This will result in a final VCF file in the directory provided to -o, or by default in your current directory: ./LINE1.final_comp.vcf.




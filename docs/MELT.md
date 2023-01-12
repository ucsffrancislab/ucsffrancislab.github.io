
#	MELT

https://melt.igs.umaryland.edu


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




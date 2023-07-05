
#	STAR

The preferred aligner for RNA, given that its reference is built from sequence and a feature files.

[STAR](https://github.com/alexdobin/STAR) (good for RNASeq if index created with gtf)



##	A standardized pipeline

https://docs.gdc.cancer.gov/Data/Bioinformatics_Pipelines/Expression_mRNA_Pipeline/#rna-seq-alignment-workflow


Create standard reference
```
STAR
--runMode genomeGenerate
--genomeDir <star_index_path>
--genomeFastaFiles <reference>
--sjdbOverhang 100 - this is default. why bother specifying?
--sjdbGTFfile <gencode.v36.annotation.gtf>
--runThreadN 8
```

Align sample but don't create bam file? Just the sample_name.SJ.out.tag
```
STAR
--genomeDir <star_index_path>
--readFilesIn <fastq_left_1>,<fastq_left2>,... <fastq_right_1>,<fastq_right_2>,...
--runThreadN <runThreadN>
--outFilterMultimapScoreRange 1
--outFilterMultimapNmax 20
--outFilterMismatchNmax 10
--alignIntronMax 500000
--alignMatesGapMax 1000000
--sjdbScore 2
--alignSJDBoverhangMin 1
--genomeLoad NoSharedMemory
--readFilesCommand <bzcat|cat|zcat>
--outFilterMatchNminOverLread 0.33
--outFilterScoreMinOverLread 0.33
--sjdbOverhang 100
--outSAMstrandField intronMotif
--outSAMtype None
--outSAMmode None
```

Create a reference for each sample using said SJ.out.tab?
```
STAR
--runMode genomeGenerate
--genomeDir <output_path>
--genomeFastaFiles <reference>
--sjdbOverhang 100 - this is default. why bother specifying?
--runThreadN <runThreadN>
--sjdbFileChrStartEnd <SJ.out.tab from previous step>
```

Align again creating actual bam
```
STAR
--genomeDir <output_path from previous step>
--readFilesIn <fastq_left_1>,<fastq_left2>,... <fastq_right_1>,<fastq_right_2>,...
--runThreadN <runThreadN>
--outFilterMultimapScoreRange 1
--outFilterMultimapNmax 20
--outFilterMismatchNmax 10
--alignIntronMax 500000
--alignMatesGapMax 1000000
--sjdbScore 2
--alignSJDBoverhangMin 1
--genomeLoad NoSharedMemory
--limitBAMsortRAM 0
--readFilesCommand <bzcat|cat|zcat>
--outFilterMatchNminOverLread 0.33
--outFilterScoreMinOverLread 0.33
--sjdbOverhang 100
--outSAMstrandField intronMotif
--outSAMattributes NH HI NM MD AS XS
--outSAMunmapped Within
--outSAMtype BAM SortedByCoordinate
--outSAMheaderHD @HD VN:1.4
--outSAMattrRGline <formatted RG line provided by wrapper>
```


##	Recommended options ...

```
--runMode alignReads \
--runThreadN ${threads} \
--genomeDir ${ref} \
--readFilesType Fastx \
--readFilesIn ${R1} ${R2} \
--readFilesCommand zcat \
--outSAMstrandField intronMotif \
--outSAMattributes Standard XS \
--outSAMtype BAM SortedByCoordinate \
--outSAMattrRGline ID:${base} SM:${base} \
--outSAMunmapped Within KeepPairs \
--outFileNamePrefix ${outFileNamePrefix} 
```


Keep unmapped in BAM file.
I think that otherwise, they are dumped into a fastx file, or perhaps just dumped.
```
--outSAMunmapped Within KeepPairs \
```



It is a very good idea to have this RG tag added to the alignment file.
Some software, like GATK, require it.
```
--outSAMattrRGline ID:${base} SM:${base} \
```


Adding the XS tag is a new addition, semi-needed by TEProF2 / stringtie.
```
--outSAMstrandField intronMotif \
--outSAMattributes Standard XS \
```

If the data is "stranded", a STAR provided awk script 

```
samtools view -h BAMFILE | awk -v strType=2 -f tagXSstrandedData.awk | samtools view -h -o BAMFILE_WITH_XS_TAG -
```




##	Sorting

Sorted by coordinate is generally more desired, but your call here.
```
--outSAMtype BAM SortedByCoordinate \
```

Sorting can be memory intensive. I've between 30GB and 130GB needed.
```
EXITING because of fatal ERROR: not enough memory for BAM sorting: 
SOLUTION: re-run STAR with at least --limitBAMsortRAM 39895949651
```
Specifying a value is usually unnecessary as STAR uses what's available
although STAR will sometimes suggest it.
```
--limitBAMsortRAM $[SLURM_NTASKS*7000000000]
```
When increasing resources, I usually increase CPUs and Memory.
There doesn't appear to be a way to calculate how much memory is needed before running.
Increasing just memory to above the asking is likely a good idea.
With the increase in CPUs (threads), STAR will create additional temporary files.
However, there is a limit on the number of open files of about 1000.
To counter this increase in files, 
```
--outBAMsortingBinsN
```
must be decreased. It defaults to 50. With 16 threads, it results in 800 temporary sort files. All good.

With 32 threads, it appears to try to create 1600 and then fails with.
```
BAMoutput.cpp:27:BAMoutput: exiting because of *OUTPUT FILE* error: could not create output file /scratch/gwendt/1468442/outdir/02-2483-01A-01R-1849-01+2._STARtmp//BAMsort/5/16
SOLUTION: check that the path exists and you have write permission for this file. Also check ulimit -n and increase it to allow more open files.
```

The number of threads times the number of sorting bins needs to be less than 1000.

The 26-5134-01A-01R-1850-01+1 sample appears to require the largest amount of sorting memory should you wish to test.



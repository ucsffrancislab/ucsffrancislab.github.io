
# REdiscoverTE

https://www.nature.com/articles/s41467-019-13035-2

http://research-pub.gene.com/REdiscoverTEpaper/software/

http://research-pub.gene.com/REdiscoverTEpaper/software/REdiscoverTE_README.html

http://research-pub.gene.com/REdiscoverTEpaper/software/REdiscoverTE_1.0.1.tar.gz



```BASH
salmon index \
	-t /home/gwendt/REdiscoverTE/rollup_annotation/genome.fasta \
	--threads 64 \
	-i /francislab/data1/refs/salmon/REdiscoverTE
```





```BASH
SALMON="/francislab/data1/refs/salmon"
DIR="/francislab/data1/working/20200320_Raleigh_Meningioma_RNA/20200320-viral_expression/trimmed"

for f in ${DIR}/???.fastq.gz ; do

	echo $f

	base=${f%.fastq.gz}
	echo $base

#	$(SALMON_0.8.2_EXE)  quant  \
#		--seqBias  --gcBias  \
#		-i $(SALMON_INDEX_DIR)  \
#		-l 'A'  $(FASTQ_INPUT_ARGS)  \
#		-o $(SALMON_COUNTS_DIR)

	echo "salmon quant --seqBias --gcBias \
		--index ${SALMON}/REdiscoverTE \
		--libType A --unmatedReads ${f} \
		--validateMappings \
		-o ${base}.salmon.REdiscoverTE \
		--threads 8" | \
		qsub -l vmem=64gb \
		-N $(basename $f .fastq.gz) \
		-l nodes=1:ppn=8 \
		-j oe -o ${base}.salmon.REdiscoverTE.out.txt

done

```



```R
>a=readRDS("rmsk_annotation.RDS")

> head(a)
                               md5 n.loci   repName      repClass     repFamily
1 9bd267abb08d45904c1741db501b5bdd      1 (TAACCC)n Simple_repeat Simple_repeat
2 e98ef316f41469ed5d18179d6f4c246f      1      TAR1     Satellite          telo
3 72834a9eaede5c9cad25de2588a65486      1      MIR3          SINE           MIR
4 accd2fc2b8c6fb63f05975ebeaf89439      1        L3          LINE           CR1
5 7c0d2042077280c3b66ace0a6b875aa0      1       MIR          SINE           MIR
6 a1307bfab38d3c74e299da0da2e6cf5e      1       MIR          SINE           MIR
  selected_feature            idx
1       intergenic 1__10001_10468
2       intergenic 1__10469_11447
3           intron 1__15265_15355
4           intron 1__19972_20405
5           intron 1__23120_23371
6           intron 1__24088_24250


>names(a)
[1] "md5"              "n.loci"           "repName"          "repClass"        
[5] "repFamily"        "selected_feature" "idx"             



> a=readRDS("GENCODE.V26.Basic_Gene_Annotation_md5.RDS")
> names(a)
[1] "md5"           "n.genes"       "transcript_ID" "ensembl_ID"   
[5] "symbol"        "geneID"       
> head(a)
                               md5 n.genes   transcript_ID      ensembl_ID
1 0001d490f52254a5ada6d26ad8880585       1 ENST00000263095 ENSG00000083844
2 0002168cd59867bac9fbab470d7aecf3       1 ENST00000442825 ENSG00000109684
3 0002d0449e073ab67c8a80063a42dbf2       1 ENST00000317538 ENSG00000163867
4 000400e08761c20fdf45ece97312b29e       1 ENST00000635564 ENSG00000236963
5 0005968766852627902d3f8ea965e224       1 ENST00000352065 ENSG00000148737
6 0005a4315808f41bb2ab181627a54afb       1 ENST00000472609 ENSG00000220831
     symbol geneID
1    ZNF264   9422
2      CLNK 116449
3     ZMYM6   9204
4 LINC01141     NA
5    TCF7L2   6934
6  NDUFA5P9     NA

```























```R
BiocManager::install(c("tibble","readr","dplyr","edgeR","Biobase"),update = TRUE, ask = FALSE)
```

```BASH
./rollup.R -h
Usage: Rscript rollup.R [options]
Rollup takes 1-5 minutes per quant.sf file on a 2018 laptop for a quant.sf with 5+ million lines.
USAGE EXAMPLE:
     rollup.R --metadata=./METADATA.tsv    --datadir=./path/to/annotation_dir/  --assembly=hg38 --outdir=Rolled_up_out_dir
 or:
     rollup.R --metadata=./METADATA.tsv    --datadir=./path/to/annotation_dir/  --priorcount=1 --nozero --threads=8 --assembly=hg38 --outdir=Rolled_up_out_dir

 The METADATA.tsv (tab-delimited) file needs two columns: 'sample' and 'quant_sf_path', as in this example:
       sample         quant_sf_path                  <-- These column names are required. Capitalization is irrelevant.
       SOME_ID_1      /full/path/to/ID1/quant.sf.gz
       SOME_ID_2      /full/path/to/ID2/quant.sf.gz  <-- Rollup can handle gzipped files as well.
       SOME_ID_3      /full/path/to/ID3/quant.sf         Gzipping your quant.sf files is recommended.
 
 ABOUT THE OUTPUT FILES:
     * 'genenorm' files: CPM was calculated using the gene-level counts (lib.sizes).
         i.e., samples are normalized by gene-level counts, not the repetitive-element-level (RE-level) counts.
         We make the underlying assumption that gene-level counts are more stable than RE-level counts.
     * 'rle_normed_eset' have the same data as above, but are presented as ExpressionSet (R) objects.
 
RollUp takes one or more 'quant.sf' files from Salmon (each of which contains transcript-level counts for a single sample) and aggregates the results by repetitive element category ('repName').

Options:
	-m METADATA, --metadata=METADATA
		Input metadata .TSV file with two columns, 'quant_sf_path' and 'sample'. See '--help' for details.

	-t THREADS, --threads=THREADS
		Num processor threads to use. Default is 2.

	-a ASSEMBLY, --assembly=ASSEMBLY
		Genome assembly version. Must be 'hg38' for now.

	-d DATADIR, --datadir=DATADIR
		Directory with reference transcriptome annotation data files. This data should have been included with your 'rollup' download.

	-o OUTDIR, --outdir=OUTDIR
		The output directory for Rollup results, which will be created by this program.

	-p PRIORCOUNT, --priorcount=PRIORCOUNT
		Prior count used in edgeR::cpm(...) calculation. Default is 5.

	-v, --verbose
		Prints verbose messages. Useful for debugging.

	-z, --nozero
		Removes any elements (rows) with zero counts. Note: this causes .RDS output files to have differing numbers of rows.

	-h, --help
		Show this help message and exit


```



```BASH

DIR="/francislab/data1/working/20200320_Raleigh_Meningioma_RNA/20200320-viral_expression"
echo -e "sample\tquant_sf_path" > ${DIR}/REdiscoverTE.tsv
ls -1 ${DIR}/trimmed/*REdiscoverTE/quant.sf \
		| awk -F/ '{split($8,a,".");print a[1]"\t"$0}' \
		>> ${DIR}/REdiscoverTE.tsv

echo "/francislab/data1/refs/REdiscoverTE/rollup.R \
		--metadata=${DIR}/REdiscoverTE.tsv \
		--datadir=/francislab/data1/refs/REdiscoverTE/rollup_annotation/ \
		--nozero --threads=64 --assembly=hg38 \
		--outdir=${DIR}/REdiscoverTE_rollup/" | \
		qsub -l vmem=500gb -N rollup \
		-l nodes=1:ppn=64 \
		-j oe -o ${DIR}/REdiscoverTE_rollup.out.txt
```





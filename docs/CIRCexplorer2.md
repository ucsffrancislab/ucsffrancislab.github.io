
#	CIRCexplorer2

##	C4 Install and Test



#	CircExplorer2


https://circexplorer2.readthedocs.io/en/latest/tutorial/setup/


##	Install

```
python3 -m pip install --upgrade --user circexplorer2

```


##	Download

Download human RefSeq gene annotation file ( Downloads refFlat.txt.gz and gunzips to hg38_ref.txt )

```
fetch_ucsc.py hg38 ref hg38_ref.txt
chmod -w hg38_ref.txt refFlat.txt.gz
```

hg38 and mm10 only have RefSeq and KnownGenes (GENCODE) gene annotations, and does not support Ensembl gene annotations.


Download human KnownGenes gene annotation file ( knownGene.txt.gz and kgXref.txt.gz then creates hg38_kg.txt )

```
fetch_ucsc.py hg38 kg hg38_kg.txt
chmod -w hg38_kg.txt kgXref.txt.gz
```



Concatenate all

```
cat hg38_ref.txt hg38_kg.txt > hg38_ref_all.txt
chmod -w hg38_ref_all.txt
```


Convert gene annotation file to GTF format (require genePredToGtf from kentutils)

```
cut -f2-11 hg38_ref_all.txt | genePredToGtf file stdin hg38_ref_all.gtf
chmod -w hg38_ref_all.gtf
```




Download human reference genome sequence file chromFa.tar.gz (roughly 1GB) (983726049)
( chromFa.tar.gz opened to hg38.fa and hg38.fa.fai )
Much Faster on c4/c4-dt1 but still takes about 2.5 hours

```
fetch_ucsc.py hg38 fa hg38.fa
chmod -w chromFa.tar.gz hg38.fa hg38.fa.fai
ln -s hg38.fa bowtie1_index.fa
ln -s hg38.fa bowtie2_index.fa
```



##	Prepare Indices


Skip these steps until you understand / decide how you're going to align.


CIRCexploer2 TopHat2/TopHat-Fusion pipeline requires Bowtie and Bowtie2 index files for reference genome. You could use bowtie-build and bowtie2-build to index relevant genome. Or you could use CIRCexplorer2 align to automatically index the genome file (See Alignment).



index genome for Bowtie
```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="bowtie-build" \
 --time=20160 --nodes=1 --ntasks=64 --mem=490G --output=${PWD}/bowtie-build.out.log \
 --wrap "module load bowtie/1.3.1; bowtie-build --threads 64 /francislab/data1/refs/CIRCexplorer2/bowtie1_index.fa /francislab/data1/refs/CIRCexplorer2/bowtie1_index; chmod -w /francislab/data1/refs/CIRCexplorer2/bowtie1_index*"

```


index genome for Bowtie2
```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="bowtie2-build" \
 --time=20160 --nodes=1 --ntasks=64 --mem=490G --output=${PWD}/bowtie2-build.out.log \
 --wrap "module load bowtie2/2.4.1; bowtie2-build --threads 64 /francislab/data1/refs/CIRCexplorer2/bowtie2_index.fa /francislab/data1/refs/CIRCexplorer2/bowtie2_index; chmod -w /francislab/data1/refs/CIRCexplorer2/bowtie2_index*"

```


index genome for STAR
```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="STAR" \
 --time=20160 --nodes=1 --ntasks=64 --mem=490G --output=${PWD}/STAR-genomeGenerate.out.log \
 --wrap "module load star/2.7.7a; STAR --runMode genomeGenerate --runThreadN 64 --genomeFastaFiles /francislab/data1/refs/CIRCexplorer2/hg38.fa --sjdbGTFfile /francislab/data1/refs/CIRCexplorer2/hg38_ref_all.gtf --genomeDir /francislab/data1/refs/CIRCexplorer2/hg38-ref_all-2.7.7a; chmod -R -w /francislab/data1/refs/CIRCexplorer2/hg38-ref_all-2.7.7a"

```



##	Align



Aligning with CIRCexplorer2 takes a while. 
Seems like it creates its own indexes from these indexes and gtf
Should probably create a good index and use STAR instead.
This would also facilitate paired end alignment.
```

module load tophat/2.1.1 bowtie/1.3.1 bowtie2/2.4.1

CIRCexplorer2 align -G /francislab/data1/refs/CIRCexplorer2/hg38_ref_all.gtf \
  --bowtie1 /francislab/data1/refs/CIRCexplorer2/bowtie1_index \
  --bowtie2 /francislab/data1/refs/CIRCexplorer2/bowtie2_index \
  -f /francislab/data1/working/20200720-TCGA-GBMLGG-RNA_bam/20200803-bamtofastq/out/02-0047-01A-01R-1849-01+1_R1.fastq.gz > CIRCexplorer2_align.log



[2023-04-27 09:13:10] Beginning TopHat run (v2.1.1)
-----------------------------------------------
[2023-04-27 09:13:10] Checking for Bowtie
		  Bowtie version:	 2.4.1.0
[2023-04-27 09:13:11] Checking for Bowtie index files (genome)..
[2023-04-27 09:13:11] Checking for reference FASTA file
	Warning: Could not find FASTA file /francislab/data1/refs/bowtie2/francislab/data2/refs/CIRCexplorer2/alignment/bowtie2_index/bowtie2_index.fa
[2023-04-27 09:13:11] Reconstituting reference FASTA file from Bowtie index
  Executing: /software/c4/cbi/software/bowtie2-2.4.1/bowtie2-inspect /francislab/data2/refs/CIRCexplorer2/alignment/bowtie2_index/bowtie2_index > /francislab/data2/refs/CIRCexplorer2/alignment/tophat/tmp/bowtie2_index.fa
[2023-04-27 09:17:15] Generating SAM header for /francislab/data2/refs/CIRCexplorer2/alignment/bowtie2_index/bowtie2_index
[2023-04-27 09:17:41] Reading known junctions from GTF file
[2023-04-27 09:18:19] Preparing reads
	 left reads: min. length=76, max. length=76, 57201535 kept reads (71936 discarded)
[2023-04-27 09:35:57] Building transcriptome data files /francislab/data2/refs/CIRCexplorer2/alignment/tophat/tmp/hg38_ref_all
[2023-04-27 09:37:13] Building Bowtie index from hg38_ref_all.fa

```

Canceled as have paired data so will be using STAR anyway.

Apparently uses environment variable BOWTIE2_INDEXES to locate bowtie2 index and not the path provided.

/francislab/data1/refs/bowtie2/francislab/data2/refs/CIRCexplorer2/alignment/bowtie2_index/bowtie2_index.fa







64 failed. Trying 32 thread. Failed. Trying 16. 16 works. Guess all the extra threads results in too many open files.

```

sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="STAR" \
--time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${PWD}/STAR-align.out.log \
--wrap "module load star/2.7.7a; \
STAR --chimSegmentMin 10 --runMode alignReads \
--runThreadN 16 \
--readFilesCommand zcat \
--readFilesIn /francislab/data1/working/20200720-TCGA-GBMLGG-RNA_bam/20200803-bamtofastq/out/02-0047-01A-01R-1849-01+1_R1.fastq.gz \
/francislab/data1/working/20200720-TCGA-GBMLGG-RNA_bam/20200803-bamtofastq/out/02-0047-01A-01R-1849-01+1_R2.fastq.gz \
--genomeDir /francislab/data1/refs/CIRCexplorer2/hg38-ref_all-2.7.7a  \
--readFilesType Fastx \
--outSAMtype BAM SortedByCoordinate \
--outFileNamePrefix /francislab/data1/refs/CIRCexplorer2/02-0047-01A."

BAMoutput.cpp:27:BAMoutput: exiting because of *OUTPUT FILE* error: could not create output file /francislab/data1/refs/CIRCexplorer2/02-0047-01A._STARtmp//BAMsort/19/46
SOLUTION: check that the path exists and you have write permission for this file. Also check ulimit -n and increase it to allow more open files.

Apr 27 11:34:38 ...... FATAL ERROR, exiting

``







```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="CIRC-Parse" \
--time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${PWD}/CIRCexplorer2-parse.out.log \
--wrap "CIRCexplorer2 parse -t STAR 02-0047-01A.Chimeric.out.junction"

```








```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="CIRC-Anno" \
--time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${PWD}/CIRCexplorer2-annotate.out.log \
--wrap "CIRCexplorer2 annotate --ref ${PWD}/hg38_ref_all.txt --genome ${PWD}/hg38.fa --bed ${PWD}/back_spliced_junction.bed --output ${PWD}/circularRNA_known.txt"

```




```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="CIRC-Ass" \
--time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${PWD}/CIRCexplorer2-assemble.out.log \
--wrap "CIRCexplorer2 assemble -ref ${PWD}/hg38_ref_all.txt --tophat ${PWD}/tophat --output ${PWD}/assemble"

```





```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --job-name="CIRC-denovo" \
--time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${PWD}/CIRCexplorer2-denovo.out.log \
--wrap "CIRCexplorer2 denovo --ref ${PWD}/hg38_ref_all.txt --genome ${PWD}/hg38.fa --bed ${PWD}/back_spliced_junction.bed --abs abs --as as --tophat ${PWD}/tophat --pAplus pAplus_tophat --output ${PWD}/denovo"

```































##	TIPCC Notes (old and with python2)

This seems to require a very particular set of dependencies in a particular order.
My final set for TIPCC includes the following.
I'm not entirely confident that this is accurate as some errors did not repeat and some work was being done on the cluster at the time.
Nevertheless, ...




For some reason, without the -f option, the second command fails. The first doesn't care. Buffer size?
Don't know exactly why, but "CIRCexplorer2 align" may actually work now. Nope just failed later!!!!

```
bowtie_header_cmd += ["-f"] 

/home/gwendt/.local/bowtie-1.2.3/bowtie --sam /francislab/data1/refs/CIRCexplorer2/bowtie1_index -f /dev/null

/home/gwendt/.local/bowtie-1.2.3/bowtie --sam /francislab/data1/raw/20201127-EV_CATS/CIRCexplorer2/L6_R1/tophat_fusion/tmp/segment_juncs -f /dev/null
```




```
pip install --upgrade --user pysam==0.15.2
export PATH="$HOME/.local/tophat-2.1.0/:$PATH"
module load samtools/1.7
module load bowtie2/2.3.4.1
module load bowtie/1.2.2
module load python/2.7.10
```

Works. No idea why it failed before. So confused.





ADD COMMANDS ....





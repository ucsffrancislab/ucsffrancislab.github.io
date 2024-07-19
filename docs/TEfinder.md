
#	TEfinder

[TEfinder](https://github.com/VistaSohrab/TEfinder)


Given the limitations and difficulties, I've been creating Singularity containers for these new pipelines.


I've created a container and image (remotely) with ...

```
cd ~/github/ucsffrancislab/genomics/singularity

singularity remote login --tokenfile ~/sylabs-token 

singularity build --remote TEfinder.img TEfinder
```


I ran the following on a Friday.
```
cp /francislab/data1/refs/sources/igv.org.genomes/hg38/rmsk/hg38_rmsk_LTR.gtf

awk -F '\t' '{print $9}' hg38_rmsk_LTR.gtf  | awk -F '"' '{print $2}' | sort | uniq > hg38_rmsk_LTR.txt

export SINGULARITY_BINDPATH=/francislab

singularity exec TEfinder.img TEfinder --threads 8 -alignment /francislab/data1/raw/20200909-TARGET-ALL-P2-RNA_bam/bam/10-PAUCDY-09A-01R.bam -fa ${PWD}/hg38.fa -gtf ${PWD}/hg38_rmsk_LTR.gtf -te ${PWD}/hg38_rmsk_LTR.txt
```
I had to kill it on Monday.

It ran all weekend with many, many processes like ...
```
/bin/bash /usr/bin/TEfinder -alignment /francislab/data1/raw/20200909-TARGET-ALL-P2-RNA_bam/bam/10-PAUCDY-09A-01R.

bedtools intersect -abam TEfinder_20220218073610/Alignments.bam -b TEfinder_20220218073610/LTR17/LTR17_TE.gff -wa
```
I think that there is a lot of cross comparison. Limiting the elements of interest list could speed this up.


Also, need to find a way to skip sorting. Waste of time to sort a already-sorted bam. Can take hours.







Apparently, the references (fa and gtf) need to be in precisely the same order.

Actually, it seems that it is numeric chromosome order (1,2,3,...) not (1,10,11,...). And also seems to not use the alternates.
Included nevertheless.
No chrM in gtf file.

```
dir=/francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/latest
for i in $( seq 1 22 ) X Y Un ; do
cat ${dir}/chromosomes/chr${i}.fa
cat ${dir}/chromosomes/chr${i}_*.fa
done > ${dir}/hg38.sorted.fa
```


```
dir=${HOME}/github/ucsffrancislab/genomics/singularity/TEfinderClusterTest
mkdir ${dir}
cd ${dir}
ln -s /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/latest/hg38.sorted.fa
ln -s /francislab/data1/refs/sources/igv.org.genomes/hg38/rmsk/hg38_rmsk_LTR.gtf

cut -c4- /francislab/data1/refs/sources/igv.org.genomes/hg38/rmsk/hg38_rmsk_LTR.gtf | awk 'BEGIN{FS=OFS="\t"}{i=index($1,"_"); if(i>0){ c=substr($1,0,i-1); d=d=substr($1,i+1) }else{ c=$1;d="" };if(c=="X")c=23;if(c=="Y")c=24;if(c=="Un")c=25;print c,d,"chr"$1,$2,$3,$4,$5,$6,$7,$8,$9}' | sort -k1n,1 -k2,2 -k6n,6 | awk 'BEGIN{FS=OFS="\t"}{print $3,$4,$5,$6,$7,$8,$9,$10,$11}' > hg38_rmsk_LTR.gtf

awk -F '\t' '{print $9}' hg38_rmsk_LTR.gtf  | awk -F '"' '{print $2}' | sort | uniq | grep HERVK  > hg38_rmsk_LTR.txt

date=$( date "+%Y%m%d%H%M%S" )
sbatch --mail-user=$(tail -1 ~/.forward) --mail-type=FAIL --job-name=TEfinder --time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${dir}/TEfinder.${date}.txt --wrap "singularity exec --bind /francislab ~/github/ucsffrancislab/genomics/singularity/TEfinder.img TEfinder -intermed yes -threads 16 -alignment /francislab/data1/raw/20200909-TARGET-ALL-P2-RNA_bam/bam/10-PAUCDY-09A-01R.bam -fa ${dir}/hg38.sorted.fa -gtf ${dir}/hg38_rmsk_LTR.gtf -te ${dir}/hg38_rmsk_LTR.txt"
```







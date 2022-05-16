
#	xTea

[xTea](https://github.com/parklab/xTea)


Given the limitations and difficulties, I've been creating Singularity containers for these new pipelines.


Built with
```
singularity remote login --tokenfile ~/sylabs-token 

singularity build --remote xTea.img xTea
```

This one is tricky.

The following runs in a second.
```
export SINGULARITY_BINDPATH=/francislab

singularity exec ../xTea.img xtea -i ${PWD}/sample_id.txt -b ${PWD}/illumina_bam_list.txt -x null -p ${PWD}/tmp/ -o submit_jobs.sh -l ${PWD}/rep_lib_annotation/ -r /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/latest/hg38.fa -g /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/genes/hg38.ncbiRefSeq.gtf -f 5907 -y 15 
```
It creates a short script with a couple sbatch commands.

The commands are a bit predictable. I tried to run them in the singularity environment.
```
chmod +x /c4/home/gwendt/github/ucsffrancislab/genomics/singularity/xTeaTest/tmp/10-PAUCDY-09A-01R/L1/run_xTEA_pipeline.sh

singularity exec ../xTea.img /c4/home/gwendt/github/ucsffrancislab/genomics/singularity/xTeaTest/tmp/10-PAUCDY-09A-01R/L1/run_xTEA_pipeline.sh
```
Still working out the bugs.






```
mkdir -p /francislab/data1/refs/sources/ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_33
cd /francislab/data1/refs/sources/ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_33
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_33/gencode.v33.annotation.gff3.gz
gunzip gencode.v33.annotation.gff3.gz
```


```
dir=${HOME}/github/ucsffrancislab/genomics/singularity/xTeaClusterTest
cd ${dir}
cp ../xTeaTest/sample_id.txt ./
cp ../xTeaTest/illumina_bam_list.txt ./
ln -s ~/github/ucsffrancislab/genomics/singularity/xTeaDemo/rep_lib_annotation

singularity exec --bind /francislab ${HOME}/github/ucsffrancislab/genomics/singularity/xTea-python3.6.img xtea -i ${dir}/sample_id.txt -b ${dir}/illumina_bam_list.txt -x null -p ${dir}/tmp/ -o ${dir}/submit_jobs.sh -l ${dir}/rep_lib_annotation/ -r /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/latest/hg38.fa -g /francislab/data1/refs/sources/ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_33/gencode.v33.annotation.gff3 -f 5907 -y 15

chmod +x /c4/home/gwendt/github/ucsffrancislab/genomics/singularity/xTeaClusterTest/tmp/10-PAUCDY-09A-01R/HERV/run_xTEA_pipeline.sh

date=$( date "+%Y%m%d%H%M%S" )
sbatch --mail-user=George.Wendt@ucsf.edu --mail-type=FAIL --job-name=xTEA-HERV --time=20160 --nodes=1 --ntasks=16 --mem=120G --output=${dir}/xTea.${date}.txt --wrap "singularity exec --bind /francislab ~/github/ucsffrancislab/genomics/singularity/xTea-python3.6.img /c4/home/gwendt/github/ucsffrancislab/genomics/singularity/xTeaClusterTest/tmp/10-PAUCDY-09A-01R/HERV/run_xTEA_pipeline.sh"
```

Don't know what's going on here. 
```
grep "^Error" xTea.20220221155122.txt | uniq -c
  33472 Error happen at merge clip and disc feature step: chr5 not exist
  42758 Error happen at merge clip and disc feature step: chr7 not exist
  19482 Error happen at merge clip and disc feature step: chr9 not exist
  21590 Error happen at merge clip and disc feature step: chr10 not exist
  18488 Error happen at merge clip and disc feature step: chr15 not exist
   9883 Error happen at merge clip and disc feature step: chr18 not exist
  13475 Error happen at merge clip and disc feature step: chr20 not exist
  18601 Error happen at merge clip and disc feature step: chr22 not exist
```
Likely a failure. Many empty files.
```
ll -tr tmp/10-PAUCDY-09A-01R/HERV
total 58938
-rw-r----- 1 gwendt francislab       17 Feb 21 15:51 sample_id.txt
-rwxr-x--- 1 gwendt francislab     2321 Feb 21 15:51 run_xTEA_pipeline.sh
-rw-r----- 1 gwendt francislab       88 Feb 21 15:51 bam_list.txt
-rw-r----- 1 gwendt francislab       97 Feb 21 15:51 bam_list1.txt
-rw-r----- 1 gwendt francislab 43158154 Feb 21 21:00 candidate_list_from_clip.txt_tmp
-rw-r----- 1 gwendt francislab 15982931 Feb 21 21:00 candidate_list_from_clip.txt
-rw-r----- 1 gwendt francislab  1200138 Feb 21 22:45 candidate_list_from_disc.txt.clip_sites_raw_disc.txt
-rw-r----- 1 gwendt francislab     4734 Feb 21 22:45 candidate_list_from_disc.txt
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt.high_confident
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt.gntp.features0.out
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt.gntp.features
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt.before_filtering
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt.before_calling_transduction.sites_cov
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt.before_calling_transduction
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns.txt
drwxr-x--- 8 gwendt francislab       14 Feb 21 23:16 tmp
-rw-r----- 1 gwendt francislab        0 Feb 21 23:16 candidate_disc_filtered_cns_with_gene.txt
```
Those chrs are in the clip file by not the disc file. Doesn't really look like a problem.

Somehow the filtering isn't keeping anything. Expected? Or fail?





So this really only works on DNA.
Ran on NA12878 DNA `/.../CEPH-ENA-PRJEB3381/20220406-xTea-Demo/`  for HERV, Alu, SVA, and L1.
Seemed to work fine.






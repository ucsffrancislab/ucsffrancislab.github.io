
#	xTea

[xTea](https://github.com/VistaSohrab/TEfinder)


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





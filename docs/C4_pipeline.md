
#	C4 Sample Pipeline




For example, a preprocessing script (still in progress)

Note: use --parsable so that sbatch returns only the job id to use as a dependency



/francislab/data1/working/20210205-EV_CATS/20210205-preprocessing/preprocess.bash 


```BASH
#!/usr/bin/env bash
#SBATCH --export=NONE		#	required if using module

module load CBI
module load star/2.7.7a


#/francislab/data1/raw/20210205-EV_CATS/SFHH001A_S1_L001_R1_001.fastq.gz
#/francislab/data1/raw/20210205-EV_CATS/SFHH001B_S2_L001_R1_001.fastq.gz
#/francislab/data1/raw/20210205-EV_CATS/Undetermined_S0_L001_R1_001.fastq.gz


#	uses python3 so need to run on C4

mkdir -p ${PWD}/output

for fastq in /francislab/data1/raw/20210205-EV_CATS/*.fastq.gz ; do

	basename=$( basename $fastq .fastq.gz )

	copy_umi_id=""
	f=${PWD}/output/${basename}_w_umi.fastq.gz
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		copy_umi_id=$( sbatch --parseable --job-name=copy_umi_${basename} --time=60 --ntasks=2 --mem=15G \
			--output=${PWD}/output/${basename}.copy_umi.output.txt \
			${PWD}/copy_umi.bash --threads 10 --umi-length 12 -i ${fastq} -o ${f} )
	fi

	depend=""
	cutadapt_id=""
	f=${PWD}/output/${basename}_w_umi.trimmed.fastq.gz
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi
		#	gres=scratch should be about total needed divided by num threads
		cutadapt_id=$( sbatch ${depend} --parseable --job-name=cutadapt_${basename} --time=60 --ntasks=2 --mem=15G \
			--output=${PWD}/output/${basename}.cutadapt.output.txt \
			${PWD}/cutadapt.bash --trim-n --match-read-wildcards -u 16 -n 3 \
				-a AGATCGGAAGAGCACACGTCTG -a AAAAAAAA -m 15 \
				-o ${f} ${PWD}/output/${basename}_w_umi.fastq.gz
	fi

	depend=""
	f=${PWD}/output/${basename}_w_umi.trimmed.STAR.Aligned.sortedByCoord.out.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi
		sbatch ${depend} --job-name=${basename} --time=480 --ntasks=8 --mem=32G \
			--output=${PWD}/output/${basename}.sbatch.STAR.output.txt \
			~/.local/bin/STAR.bash --runThreadN 8 --readFilesCommand zcat \
				--genomeDir /francislab/data1/refs/STAR/hg38-golden-ncbiRefSeq-2.7.7a-49/ \
				--sjdbGTFfile /francislab/data1/refs/fasta/hg38.ncbiRefSeq.gtf \
				--sjdbOverhang 49 \
				--readFilesIn ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz \
				--quantMode TranscriptomeSAM \
				--quantTranscriptomeBAMcompression -1 \
				--outSAMtype BAM SortedByCoordinate \
				--outSAMunmapped Within \
				--outFileNamePrefix ${PWD}/output/${basename}_w_umi.trimmed.STAR.
	fi

	depend=""
	f=${PWD}/output/${basename}_w_umi.trimmed.bowtie2phiX.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi
		sbatch ${depend} --job-name=${basename} --time=480 --ntasks=8 --mem=32G \
			--output=${PWD}/output/${basename}.bowtie2phiX.output.txt \
			~/.local/bin/bowtie2.bash --threads 8 -x /francislab/data1/refs/bowtie2/phiX \
			--very-sensitive-local -U ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz -o ${f}
	fi

	depend=""
	f=${PWD}/output/${basename}_w_umi.trimmed.STAR.mirna.Aligned.sortedByCoord.out.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi
		sbatch ${depend} --job-name=${basename} --time=480 --ntasks=8 --mem=32G \
			--output=${PWD}/output/${basename}_w_umi.trimmed.STAR.mirna.output.txt \
			~/.local/bin/STAR.bash --runThreadN 8 --readFilesCommand zcat \
				--genomeDir /francislab/data1/refs/STAR/human_mirna \
				--readFilesIn ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz \
				--outSAMtype BAM SortedByCoordinate \
				--outSAMunmapped Within \
				--outFileNamePrefix ${PWD}/output/${basename}_w_umi.trimmed.STAR.mirna.
	fi

	depend=""
	f=${PWD}/output/${basename}_w_umi.trimmed.bowtie2.mirna.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi
		sbatch ${depend} --job-name=${basename} --time=480 --ntasks=8 --mem=32G \
			--output=${PWD}/output/${basename}.bowtie2.mirna.output.txt \
			~/.local/bin/bowtie2.bash --threads 8 -x /francislab/data1/refs/bowtie2/human_mirna \
			--very-sensitive-local -U ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz -o ${f}
	fi

	depend=""
	f=${PWD}/output/${basename}_w_umi.trimmed.bowtie2.mirna.all.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi
		sbatch ${depend} --job-name=${basename} --time=480 --ntasks=8 --mem=32G \
			--output=${PWD}/output/${basename}.bowtie2.mirna.all.output.txt \
			~/.local/bin/bowtie2.bash --all --threads 8 -x /francislab/data1/refs/bowtie2/human_mirna \
			--very-sensitive-local -U ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz -o ${f}
	fi

	depend=""
	f=${PWD}/output/${basename}_w_umi.trimmed.bowtie2.hg38.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		sbatch ${depend} --job-name=${basename} --time=480 --ntasks=8 --mem=32G \
			--output=${PWD}/output/${basename}.bowtie2.hg38.output.txt \
			~/.local/bin/bowtie2.bash --threads 8 -x /francislab/data1/refs/bowtie2/hg38 \
			--very-sensitive-local -U ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz -o ${f}
	fi

	depend=""
	base=${PWD}/output/${basename}_w_umi.trimmed.bowtie2.hg38.all
	f=${base}.bam
	if [ -f $f ] && [ ! -w $f ] ; then
		echo "Write-protected $f exists. Skipping."
	else
		if [ ! -z ${cutadapt_id} ] ; then
			depend="-W depend=afterok:${cutadapt_id}"
		else
			depend=""
		fi

		threads=32
		fasta_size=$( stat --dereference --format %s ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz )
		#r2_size=$( stat --dereference --format %s ${r2} )
		#index_size=$( du -sb ${index} | awk '{print $1}' )
		#scratch=$( echo $(( ((${r1_size}+${r2_size}+${index_size})/${threads}/1000000000*11/10)+1 )) )
		# Add 1 in case files are small so scratch will be 1 instead of 0.
		# 11/10 adds 10% to account for the output

		scratch=$( echo $(( ((${fasta_size}*10)/${threads}/1000000000*11/10)+1 )) )

		echo "Using scratch:${scratch}"

		sbatch ${depend} --job-namee${basename} --time=1920 --ntasks=${threads} --mem=250G \
			--gres=scratch:${scratch}G --output=${base}.output.txt \
			~/.local/bin/bowtie2_scratch.bash --all --threads ${threads} -x /francislab/data1/refs/bowtie2/hg38 \
			--very-sensitive-local -U ${PWD}/output/${basename}_w_umi.trimmed.fastq.gz -o ${f}
	fi

done
```

[C4_scratch](/docs/C4_scratch)



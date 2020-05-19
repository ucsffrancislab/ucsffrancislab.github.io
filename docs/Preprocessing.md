
#	Preprocessing


Basic preprocessing ...

Prior to this, could / would do demultiplexing and umi tagging and consolidation with 
Demultiplexing
<https://github.com/grenaud/deML>
<https://github.com/ucsffrancislab/umi>
UMI Tagging / Consolidation
<https://github.com/ucsffrancislab/umi>



Setup

```BASH
REFS=/francislab/data1/refs
FASTA=${REFS}/fasta
date=$( date "+%Y%m%d%H%M%S" )

base=${R1%_R1.*}
R2=${R1/_R1/_R2}

base=$( basename ${base} )
echo $base

mkdir -p ${OUT}/trimmed/length/unpaired

outbase="${OUT}/trimmed/${base}"
```

Trim off adapters and filter on read quality.

Could also use cutadapt

```BASH
bbduk.bash \
	-Xmx16g \
	in1=${R1} \
	in2=${R2} \
	out1=${outbase}_R1.fastq.gz \
	out2=${outbase}_R2.fastq.gz \
	outs=${outbase}_S.fastq.gz \
	ref=${FASTA}/illumina_adapters.fa \
	ktrim=r \
	k=23 \
	mink=11 \
	hdist=1 \
	tbo \
	ordered=t \
	bhist=${outbase}.bhist.txt \
	qhist=${outbase}.qhist.txt \
	gchist=${outbase}.gchist.txt \
	aqhist=${outbase}.aqhist.txt \
	lhist=${outbase}.lhist.txt \
	gcbins=auto \
	maq=10 \
	qtrim=w trimq=5 minavgquality=0
```

Create some histogram data on the results.


```BASH
read_length_hist.bash ${outbase}_R1.fastq.gz
read_length_hist.bash ${outbase}_R2.fastq.gz
read_length_hist.bash ${outbase}_S.fastq.gz

inbase="${outbase}"
outbase="${OUT}/trimmed/length/${base}"
```

Filter out read pairs that are no longer the same length.

```BASH
filter_paired_fastq_on_equal_read_length.bash \
	${inbase}_R1.fastq.gz \
	${inbase}_R2.fastq.gz \
	${outbase}_R1.fastq.gz \
	${outbase}_R2.fastq.gz \
	${outbase}_R1_diff.fastq.gz \
	${outbase}_R2_diff.fastq.gz

read_length_hist.bash ${outbase}_R1.fastq.gz
read_length_hist.bash ${outbase}_R2.fastq.gz
read_length_hist.bash ${outbase}_R1_diff.fastq.gz
read_length_hist.bash ${outbase}_R2_diff.fastq.gz
```

Merge both reads into a single file, should that be desired.

```BASH
inbase="${outbase}"
outbase="${OUT}/trimmed/length/unpaired/${base}"

unpair_fastqs.bash -o ${outbase}.fastq.gz ${inbase}_R?.fastq.gz
```



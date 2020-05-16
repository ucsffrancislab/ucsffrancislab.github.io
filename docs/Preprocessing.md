
#	Preprocessing


Basic preprocessing ...



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

read_length_hist.bash ${outbase}_R1.fastq.gz
read_length_hist.bash ${outbase}_R2.fastq.gz
read_length_hist.bash ${outbase}_S.fastq.gz

inbase="${outbase}"
outbase="${OUT}/trimmed/length/${base}"

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

inbase="${outbase}"
outbase="${OUT}/trimmed/length/unpaired/${base}"

unpair_fastqs.bash -o ${outbase}.fastq.gz ${inbase}_R?.fastq.gz
```




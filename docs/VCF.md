
#	VCF





##	How to create



```
#!/usr/bin/env bash
#SBATCH --export=NONE   # required when using 'module'

hostname
echo "Slurm job id:${SLURM_JOBID}:"
date

#		NOPE. Will be "slurm_script" at runtime
if [ -n "${SLURM_JOB_NAME}" ] ; then
	script=${SLURM_JOB_NAME}
else
	script=$( basename $0 )
fi

#	PWD preserved by slurm for where job is run? I guess so.
arguments_file=${PWD}/${script}.arguments

if [ -n "${SLURM_ARRAY_TASK_ID}" ] ; then

	set -e	#	exit if any command fails
	set -u	#	Error on usage of unset variables
	set -o pipefail
	if [ -n "$( declare -F module )" ] ; then
		echo "Loading required modules"
		module load CBI bcftools/1.15.1
	fi
	set -x	#	print expanded command before executing it

	line=${SLURM_ARRAY_TASK_ID:-1}
	echo "Running line :${line}:"

	#	Use a 1 based index since there is no line 0.

	args=$( sed -n "$line"p ${arguments_file} )
	echo $args

	if [ -z "${args}" ] ; then
		echo "No line at :${line}:"
		exit
	fi

	echo $args




	dir=${PWD}/out
	bam=${dir}/${args}.quality.umi.t1.t3.hg38.bam
	basename=$( basename ${bam} .bam )

	bcftools mpileup -Ou -f /francislab/data1/refs/sources/hgdownload.cse.ucsc.edu/goldenPath/hg38/bigZips/latest/hg38.chrXYM_alts.fa ${bam} | bcftools call -mv -Oz -o ${dir}/${basename}.vcf.gz

	chmod a-w ${dir}/${basename}.vcf.gz


	bcftools index ${dir}/${basename}.vcf.gz

	chmod a-w ${dir}/${basename}.vcf.gz.csi


else

	date=$( date "+%Y%m%d%H%M%S" )

	cat <<- EOF > ${arguments_file}
SFHH011AC
SFHH011BB
SFHH011BZ
SFHH011CH
SFHH011I
SFHH011S
SFHH011Z
SFHH011BO
	EOF

	max=$( cat ${arguments_file} | wc -l )

	mkdir ${PWD}/logs/
	date=$( date "+%Y%m%d%H%M%S%N" )
	sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL \
		--array=1-${max}%1 --job-name=${script} \
		--output="${PWD}/logs/${script}.${date}.%A_%a.out" \
		--time=1440 --nodes=1 --ntasks=8 --mem=60G \
		$( realpath ${0} )

fi

```



```
#module load CBI bcftools/1.11

#	Can fail on weird CIGAR strings.
#	bcftools: sam.c:3948: resolve_cigar2: Assertion `k < c->n_cigar' failed.
#	use module load bcftools/1.10.2 until 1.12 is released
```




##	How to compare





```
for a in out/*.vcf.gz; do echo $a
for b in out/*.vcf.gz; do echo $b
if [ ${a} != ${b} ] ; then
bcftools isec -n =2 ${a} ${b} | wc -l > ${a}.$(basename ${b}).shared_isec_count
fi
done ; done

for a in out/*.vcf.gz; do echo $a
for b in out/*.vcf.gz; do echo $b
if [ ${a} != ${b} ] ; then
bcftools isec -n -1 ${a} ${b} | wc -l > ${a}.$(basename ${b}).diff_isec_count
fi
done ; done
```









They can be merged into 1 giant vcf file or compared with something like ...
`bedtools intersect ...`
 or
`bcftools isec ...`


#	put in decreasing order. WHY?

```
nohup bcftools merge -o 120207.vcf.gz -Oz newvcf/120207.100.vcf.gz newvcf/120207.80?.vcf.gz newvcf/120207.60?.vcf.gz newvcf/120207.50?.vcf.gz &
nohup bcftools merge -o 122997.vcf.gz -Oz newvcf/122997.100.vcf.gz newvcf/122997.80?.vcf.gz newvcf/122997.60?.vcf.gz newvcf/122997.50?.vcf.gz &
nohup bcftools merge -o 186069.vcf.gz -Oz newvcf/186069.100.vcf.gz newvcf/186069.80?.vcf.gz newvcf/186069.60?.vcf.gz newvcf/186069.50?.vcf.gz &
```


```BASH
for f in newvcf/*.*.vcf.gz; do echo $f ; bcftools view --no-header $f | wc -l > $f.count ; done

for f in newvcf/*.*.vcf.gz; do echo $f ; bcftools isec -C ${f%%.*}.100.vcf.gz ${f} | wc -l > $f.100_isec_count ; done
for f in newvcf/*.*.vcf.gz; do echo $f ; bcftools isec -C ${f} ${f%%.*}.100.vcf.gz | wc -l > $f.isec_100_count ; done



for f in newvcf/*.*.vcf.gz; do echo $f ; bcftools isec -n =2 ${f} ${f%%.*}.100.vcf.gz | wc -l > $f.shared_100_isec_count ; done
for f in newvcf/*.*.vcf.gz; do echo $f ; bcftools isec -n -1 ${f} ${f%%.*}.100.vcf.gz | wc -l > $f.diff_100_isec_count ; done


```







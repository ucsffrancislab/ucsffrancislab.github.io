
#	GWAS


Use plink 1.9???

Newer versions produce different output.



Using 1000genomes, for example


```
aws s3 sync s3://1000genomes/release/20130502/ /francislab/data1/raw/1000genomes/release/20130502/ --exclude "*supporting/*"


Sample,Family ID,Population,Population Description,Gender,Relationship,Unexpected Parent/Child ,Non Paternity,Siblings,Grandparents,Avuncular,Half Siblings,Unknown Second Order,Third Order,Other Comments
HG00096,HG00096,GBR,British in England and Scotland,male,,,,,,,,,,
HG00097,HG00097,GBR,British in England and Scotland,female,,,,,,,,,,
HG00098,HG00098,GBR,British in England and Scotland,male,,,,,,,,,,
HG00099,HG00099,GBR,British in England and Scotland,female,,,,,,,,,,
HG00100,HG00100,GBR,British in England and Scotland,female,,,,,,,,,,
HG00101,HG00101,GBR,British in England and Scotland,male,,,,,,,,,,

Population Description	Population Code	Super Population	DNA from Blood	Offspring available from trios	Pilot Samples	Phase1 Samples	Final Phase Samples	Total
Chinese Dai in Xishuangbanna, China	CDX	EAS	no	yes	0	0	99	99
Han Chinese in Bejing, China	CHB	EAS	no	no	91	97	103	106
Japanese in Tokyo, Japan	JPT	EAS	no	no	94	89	104	105

Description	Population Code
East Asian	EAS
South Asian	SAS
African	AFR
```

Create own metadata with Sample, Gender, Population, Population Description, Super Population, Super Population Description
by merging 20130606_sample_info\ -\ Sample\ Info.csv, 20131219.populations.tsv and 20131219.superpopulations.tsv


```
awk -F"\t" '( ARGIND == 1 ){FS="\t";s[$2]=$1} ( ARGIND == 2 ){FS="\t";p[$2]=$3; print($1,$2,$3,s[$3])}' \
	20131219.superpopulations.tsv 20131219.populations.tsv


echo -e "Sample\tFamily\tGender\tPopulation\tPopulation Description\tSuper Population\tSuper Population Description" > metadata.tsv
awk '( ARGIND == 1 ){FS="\t";s[$2]=$1} ( ARGIND == 2 ){FS="\t";p[$2]=$3} ( ARGIND == 3 ){FPAT="([^,]*)|(\"[^\"]+\")";OFS="\t"; print( $1, $2, $5, $3, $4, p[$3], s[p[$3]]) } ' 20131219.superpopulations.tsv 20131219.populations.tsv 20130606_sample_info\ -\ Sample\ Info.csv | tail -n +2 | sed 's/"//g' >> metadata.tsv








#
#	1=male, 2=female, 0=unknown
#
#	Use sample id as family id

awk 'BEGIN{FS=OFS="\t"}(NR>1){ s=($3=="male")?1:2; print $1,$1,s >> $6"_ids.txt" }' \
	/francislab/data1/raw/1000genomes/metadata.tsv

awk 'BEGIN{FS=OFS="\t"}(NR>1){ s=($3=="male")?1:2; print $1,$1,s >> $6"_"$4"_ids.txt" }' \
	/francislab/data1/raw/1000genomes/metadata.tsv


```




```BASH
#!/usr/bin/env bash

date=$( date "+%Y%m%d%H%M%S" )

basedir=/francislab/data1/raw/1000genomes/gwas
dir=${basedir}/20200402

for population in afr amr eas eur sas ; do
	echo $population
	outdir="${dir}/${population}"
	mkdir -p "${outdir}"

for vcf in /francislab/data1/raw/1000genomes/release/20130502/ALL.chr*vcf.gz ; do
	echo $vcf
	base=$( basename $vcf )
	base=${base%.phase3*}
	base=${base#ALL.}
	echo $base

	#	-l nodes=1:ppn=${threads} -l vmem=${vmem}gb \

	outbase="${outdir}/${base}.plink-make-bed"

	qsub -N ${population}:${base} \
		-o ${outbase}.${date}.out.txt -e ${outbase}.${date}.err.txt \
		~/.local/bin/plink.bash \
		-F "--check ${outdir}/${base}.bed \
			--make-bed \
			--keep ${basedir}/${population^^}_ids.txt \
			--update-sex ${basedir}/${population^^}_ids.txt \
			--vcf ${vcf} \
			--out ${outdir}/${base}"

done ; done

```








```BASH
#!/usr/bin/env bash

hostname

set -e	#	exit if any command fails
set -u	#	Error on usage of unset variables
set -o pipefail

set -x

plot_prefix=""
while [ $# -gt 0 ] ; do
	case $1 in
		--out)
			shift; out=$1; shift;;
		--pheno)
			shift; pheno=$1; shift;;
		--covar)
			shift; covar=$1; shift;;
		--beddir)
			shift; beddir=$1; shift;;
		--plot_prefix)
			shift; plot_prefix=$1; shift;;
	esac
done

## 0. Create job-specific scratch folder that ...
SCRATCH_JOB=/scratch/$USER/job/$PBS_JOBID
mkdir -p $SCRATCH_JOB
##    ... is automatically removed upon exit
##    (regardless of success or failure)
trap "{ cd /scratch/; chmod -R +w $SCRATCH_JOB/; \rm -rf $SCRATCH_JOB/ ; }" EXIT


pheno_name=$(basename ${pheno} )

f="${out}/${pheno_name}.for.manhattan.plot.png"
if [ -f $f ] && [ ! -w $f ] ; then
	echo "Write-protected $f exists. Skipping."
else
	echo "Creating $f"

	mkdir -p ${SCRATCH_JOB}/bed
	cp --archive ${beddir}/*.{bed,bim,fam} ${SCRATCH_JOB}/bed/
	scratch_beddir=${SCRATCH_JOB}/bed/

	cp --archive ${pheno} ${SCRATCH_JOB}/
	scratch_pheno=${SCRATCH_JOB}/${pheno_name}

	cp --archive ${covar} ${SCRATCH_JOB}/
	scratch_covar=${SCRATCH_JOB}/$( basename ${covar} )

	scratch_out=${SCRATCH_JOB}/out
	mkdir -p ${scratch_out}

	for bedfile in ${scratch_beddir}/*.bed ; do

		echo $bedfile

		# drop the shortest suffix match to ".*" (the .bed extension)
		bedfile_noext=${bedfile%.bed}

		#	drop the longest prefix match to "*/" (the path)
		bedfile_core=${bedfile_noext##*/}

#		mkdir -p ${OUT}/${population}/${pheno_dir}

		#	Initial run was with PLINK v1.90b3.38 64-bit (7 Jun 2016)
		#	This output produced .no.covar.assoc.logistic
		#	This is different than now with different columns
		#	Not sure how to resolve. Will try downloading older version.

		#	$ head test.no.covar.assoc.logistic 
		#	 CHR            SNP         BP   A1       TEST    NMISS         OR         STAT            P 
		#	   6    rs561313667      63979    T        ADD      489         NA           NA           NA
		#	   6    rs530120680      63980    G        ADD      489         NA           NA           NA
		#	   6    rs540888038      73938    G        ADD      489         NA           NA           NA

		#	head gwas/pheno_files_1/10298_Human_alphaherpesvirus_1.ALL.chr13.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.afr.pruned.no.covar.PHENO1.glm.logistic
		#	#CHROM	POS	ID	REF	ALT	A1	TEST	OBS_CT	OR	LOG(OR)_SE	Z_STAT	P
		#	13	19020047	rs186129910	A	T	T	ADD	661	NA	NA	NA	NA
		#	13	19020078	rs527875342	C	T	T	ADD	661	NA	NA	NA	NA
		#	13	19020095	rs140871821	C	T	T	ADD	661	1.78567	0.259021	2.2384	0.0251947
		#	13	19020165	rs550529448	T	A	A	ADD	661	1.25648	1.21356	0.188137	0.85077


		#plink2 --covar-variance-standardize \

#--out /scratch/gwendt/job/1775728.cclc01.som.ucsf.edu/pheno_files_1/NC_000898_e2e.chr1.no.covar
#Error: Failed to open /scratch/gwendt/job/1775728.cclc01.som.ucsf.edu/pheno_files_1/NC_000898_e2e.chr1.no.covar.log.  Try changing the --out parameter.

		plink --allow-no-sex \
					--snps-only \
					--logistic hide-covar \
					--covar-name C1,C2,C3,C4,C5,C6 \
					--bfile ${bedfile_noext} \
					--pheno ${scratch_pheno} \
					--out ${scratch_out}/${pheno_name}.${bedfile_core}.no.covar \
					--covar ${scratch_covar}

					#	plink  produces .assoc.logistic and .log
					#	plink2 produces .PHENO1.glm.logistic and .log

		if [ -f ${scratch_out}/${pheno_name}.${bedfile_core}.no.covar.assoc.logistic ] ; then
			#	PLINK v1.90b6.16 64-bit (19 Feb 2020)
			awk '{print $1,$2,$3,$9,$4,$7}' \
				${scratch_out}/${pheno_name}.${bedfile_core}.no.covar.assoc.logistic \
				> ${scratch_out}/${pheno_name}.${bedfile_core}.for.plot.txt
		else
			touch ${scratch_out}/${pheno_name}.${bedfile_core}.for.plot.txt
		fi

		#	plink2 output
		#	PLINK v2.00a2LM 64-bit Intel (10 Aug 2019)
		#	Newer version uses different columns in different file
		#	awk '{print $1,$3,$2,$12,$6,$9}' \
		#		${OUT}/${population}/${pheno_dir}/${pheno_name}.${bedfile_core}.no.covar.PHENO1.glm.logistic \
		#		> ${OUT}/${population}/${pheno_dir}/${pheno_name}.${bedfile_core}.for.plot.txt

	done	#	for bedfile in ${REFS}/pruned_vcfs/${population}/ALL.chr*.bed ; do

	#	Not keeping for.plot.all.txt so doesn't need a header
	#echo "CHR SNP BP P A1 OR" > ${pheno_name}.for.plot.all.txt
	#grep -v "CHR" *.for.plot.txt >> ${pheno_name}.for.plot.all.txt
	#grep --invert-match --no-filename "CHR" *.for.plot.txt >> ${pheno_name}.for.plot.all.txt
	grep --invert-match --no-filename "CHR" \
		${scratch_out}/${pheno_name}.*.for.plot.txt \
		| sed -e 's/^X/23/' -e 's/^Y/24/' \
		> ${scratch_out}/${pheno_name}.for.plot.all.txt

	#	No wildcards, so don't need to specify --no-filename
	grep --invert-match "NA" ${scratch_out}/${pheno_name}.for.plot.all.txt \
		| shuf -n 200000 > ${scratch_out}/${pheno_name}.for.qq.plot
	#	If not keeping header, don't need to skip the first line anymore!
	#tail -n +2 ${pheno_name}.for.plot.all.txt | grep -v "NA" | shuf -n 200000 > ${pheno_name}.for.qq.plot

	#	Keeping the NA rows now
	grep "NA" ${scratch_out}/${pheno_name}.for.plot.all.txt \
		> ${scratch_out}/${pheno_name}.NA.txt

	awk '$4 < 0.10' ${scratch_out}/${pheno_name}.for.plot.all.txt \
		> ${scratch_out}/${pheno_name}.for.manhattan.plot

	if [ -s ${scratch_out}/${pheno_name}.for.manhattan.plot ] && \
		[ -s ${scratch_out}/${pheno_name}.for.qq.plot ] ; then
		manhattan_qq_plot.r \
			--plot_prefix "${plot_prefix}" \
			--manhattan ${scratch_out}/${pheno_name}.for.manhattan.plot \
			--qq ${scratch_out}/${pheno_name}.for.qq.plot \
			--outpath ${scratch_out}
		#	produces ${pheno_name}.for.manhattan.plot.png
	fi

	chmod a-w ${scratch_out}/*
	mkdir -p ${out}/	#	just in case
	mv --update ${scratch_out}/* ${out}/

fi
```






Learn GWAS / Manhattan plots
https://en.wikipedia.org/wiki/Genome-wide_association_study
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6001694/
https://elifesciences.org/articles/32920
http://bioinformatics.org.au/ws09/presentations/Day3_JStankovich.pdf
Try Hail or other for GWAS?




#	Manta / Strelka


Illumina's tumor / normal variant callers.

[Structural variant and indel caller for mapped sequencing data](https://github.com/Illumina/manta)

Will require indexes for all input `bam`, `fasta` and `vcf` files to be present.

```BASH
${MANTA_INSTALL_PATH}/bin/configManta.py \
	--normalBam HCC1187BL.bam \
	--tumorBam HCC1187C.bam \
	--referenceFasta ${UNZIPPED_HG_REFERENCE_FASTA} \
	--exome \
	--runDir ${MANTA_ANALYSIS_PATH}

${MANTA_ANALYSIS_PATH}/runWorkflow.py --jobs=${PBS_NUM_PPN} --memGb=${memGb} --mode=local
```

[Strelka2 germline and somatic small variant caller](https://github.com/Illumina/strelka)

```BASH
${STRELKA_INSTALL_PATH}/bin/configureStrelkaSomaticWorkflow.py \
	--normalBam HCC1187BL.bam \
	--tumorBam HCC1187C.bam \
	--referenceFasta ${UNZIPPED_HG_REFERENCE_FASTA} \
	--indelCandidates ${MANTA_ANALYSIS_PATH}/results/variants/candidateSmallIndels.vcf.gz \
	--exome \
	--runDir ${STRELKA_ANALYSIS_PATH}

${STRELKA_ANALYSIS_PATH}/runWorkflow.py --jobs=${PBS_NUM_PPN} --memGb=${memGb} --mode=local
```

NOTE: Don't install strelka and manta in the same dir like `~/.local` as they contain some files of the same name but different content or versions. Install `manta` in `~/.local/manta` and `strelka` in `~/.local/strelka`. Your life will be much better.





After this, I select the mutations from the resulting `vcf` file with something like ...

```BASH
for f in out/*.strelka/results/variants/somatic.snvs.vcf.gz ; do
s=${f%.strelka/results/variants/somatic.snvs.vcf.gz}
s=$( basename $s )
echo $f $s
bcftools query -f "%LINE" -i 'FILTER="PASS" && TYPE="SNP" && MQ>40 && ReadPosRankSum>-8' ${f} | awk -v s=$s 'BEGIN{FS=OFS="\t"}{
split($10,normal,":")
split(normal[5],na,",")
split(normal[6],nc,",")
split(normal[7],ng,",")
split(normal[8],nt,",")
split($11,tumor,":")
split(tumor[5],ta,",")
split(tumor[6],tc,",")
split(tumor[7],tg,",")
split(tumor[8],tt,",")
if(normal[1] > 5 && tumor[1] > 5 ){
n["A"]=na[1];n["C"]=nc[1];n["G"]=ng[1];n["T"]=nt[1]
t["A"]=ta[1];t["C"]=tc[1];t["G"]=tg[1];t["T"]=tt[1]
maxin=0;for(i in n){maxin=(n[i]>n[maxin] ? i : maxin)}
delete t[maxin]
maxit=0;for(i in t){maxit=(t[i]>t[maxit] ? i : maxit)}
print $1,$2,maxin,maxit,s >> "mut.txt"
print $1,$2,maxin,maxit,"+",s >> "count_trinuc_input.csv"
}else{#DP TOO LOW FOR ONE
print s,$0 >> "TooLowDP"
} }'
done
```

Then extract the trinuc counts with ...

```BASH
count_trinuc_muts_v8.pl pvcf /francislab/data1/refs/fasta/hg38.fa count_trinuc_input.csv > mut_all_sort.txt
```


Then search for signatures with ...


```BASH


It's a secret


```





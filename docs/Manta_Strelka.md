
#	Manta / Strelka


Illumina's tumor / normal variant callers.

[Structural variant and indel caller for mapped sequencing data](https://github.com/Illumina/manta)

```BASH
${MANTA_INSTALL_PATH}/bin/configManta.py \
--normalBam HCC1187BL.bam \
--tumorBam HCC1187C.bam \
--referenceFasta hg19.fa \
--runDir ${MANTA_ANALYSIS_PATH}


--exome


${MANTA_ANALYSIS_PATH}/runWorkflow.py -j 8
```

[Strelka2 germline and somatic small variant caller](https://github.com/Illumina/strelka)

```BASH
${STRELKA_INSTALL_PATH}/bin/configureStrelkaSomaticWorkflow.py \
--normalBam HCC1187BL.bam \
--tumorBam HCC1187C.bam \
--referenceFasta hg19.fa \
--indelCandidates ${MANTA_ANALYSIS_PATH}/results/variants/candidateSmallIndels.vcf.gz \
--runDir ${STRELKA_ANALYSIS_PATH}


--exome

${STRELKA_ANALYSIS_PATH}/runWorkflow.py -m local -j 8
```



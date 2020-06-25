
#	Aligners

[bowtie2](https://github.com/BenLangmead/bowtie2)


[STAR](https://github.com/alexdobin/STAR) (good for RNASeq if index created with gtf)


[bwa](https://github.com/lh3/bwa)


[kraken2](https://github.com/DerrickWood/kraken2) - [bracken](https://github.com/jenniferlu717/Bracken/)

Kraken's "standard" library contains archaea, bacteria, viral, human and the UniVec Core.
If human is present, it usually doesn't find others so I created "abv" which is the same as "standard" but without human.
Kraken's building process dust mask's the sequences, but does not repeat mask it.
Having run it and found a lot of BeAnn, I think that I should run repeat masker on the sequences and rebuild.

```BASH
kraken2-build --db standard-20200615 --download-taxonomy --skip-maps
kraken2-build --db standard-20200615 --download-library archaea
kraken2-build --db standard-20200615 --download-library bacteria
kraken2-build --db standard-20200615 --download-library viral
kraken2-build --db standard-20200615 --download-library UniVec_Core

#	Repeat Masker on the 4 library.fna files

kraken2-build --db standard-20200615 --build --threads 32


#	Prepare Bracken reference for each anticipated read length
#	Run multiple, but not at the same time.
#	They both create the file `database.kraken` if it doesn't exist.

bracken-build -d abv -t 32 -l 150

#	Then

bracken-build -d abv -t 32 -l 100



#	Then align with kraken with the report option and analyze the report file with bracken.

kraken2 --db ${KRAKEN_DB} --threads ${threads} --output ${f} --paired --use-names ${r1} ${r2}

bracken -d ${KRAKEN_DB} -i ${SAMPLE}.kreport -o ${SAMPLE}.bracken -r ${READ_LEN} -l ${LEVEL} -t ${THRESHOLD}
```




[kallisto](https://pachterlab.github.io/kallisto/) - [sleuth](https://pachterlab.github.io/sleuth/)

[salmon](https://github.com/COMBINE-lab/salmon)


[plast](https://github.com/PLAST-software)

[blastn/blastx](https://blast.ncbi.nlm.nih.gov/Blast.cgi)

[diamond](https://github.com/bbuchfink/diamond)

[blat](https://genome-test.gi.ucsc.edu/~kent/src/)


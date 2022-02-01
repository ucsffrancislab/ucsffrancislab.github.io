
#	Assemblers

##	Short Read / Illumina

[ABySS](docs/Abyss) https://github.com/bcgsc/abyss - de novo assembler for short, paired end reads and large genome

Velvet - old and less supported

Trinity

SOAP

Meraculous

PERGA

PE-Assembler

Forge


##	Long Read / Nanopore / PacBio



`nanopore.exact.fasta` contains a collection of reads randomly extracted from a 5mb section of hg38.



###	Canu

[Canu](https://github.com/marbl/canu)

```
canu -p SEQUENCE_PREFIX -d OUT_DIRECTORY genomeSize=5m useGrid=false -corrected -trimmed -nanopore nanopore.exact.fasta
```

###	Frye

[Frye](https://github.com/fenderglass/Flye)

```
flye --genome-size 5m --threads 8 --nano-cor nanopore.exact.fasta --out-dir OUT_DIRECTORY
```

###	Shasta

[Shasta](https://github.com/chanzuckerberg/shasta)

```
shasta --input nanopore.exact.fasta --assemblyDirectory OUT_DIRECTORY
```

###	Raven

[Raven](https://github.com/lbcb-sci/raven)



####	Testing and Comparison


[Read Simulator](https://github.com/ucsffrancislab/read_simulator) was created to generate reads to test assemblers by randomly selecting subsections of a reference.






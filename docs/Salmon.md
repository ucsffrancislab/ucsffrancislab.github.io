
#	Salmon


Using Salmon for analysis


Generating an index based on some of the RepeatMasker's libs. Extracting all human with ...


```
~/.local/RepeatMasker/util/queryRepeatDatabase.pl -species human > RepeatDatabase.fa
queryRepeatDatabase
===================
RepeatMasker Database: RepeatMaskerLib.embl
RepeatMasker Combined Database: Dfam_3.1, RepBase-20181026
Species: human ( homo sapiens )

grep -c "^>" RepeatDatabase.fa
1363
```



Make salmon index (very similar to kallisto)

```BASH
salmon index -t RepeatDatabase.fa -i RepeatDatabase_21 -k 21
salmon index -t RepeatDatabase.fa -i RepeatDatabase_31 -k 31
```




```BASH
for f in ${DIR}/???.fastq.gz ; do
	echo $f
	base=${f%.fastq.gz}
	echo $base
	for k in 21 31 ; do
		echo "salmon quant --index ${SALMON}/RepeatDatabase_${k} --libType A --unmatedReads ${f} --validateMappings -o ${base}.salmon.RepeatDatabase_${k} --threads 8" | qsub -l vmem=16gb -N $(basename $f .fastq.gz)_${k} -l nodes=1:ppn=8
	done
done
```



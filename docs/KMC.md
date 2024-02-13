
#	KMC

https://github.com/refresh-bio/KMC

Initially used through iMOKA.

KMC appears to be much faster and less resource intensive than jellyfish.

Trying to use outside of iMOKA.




Run KMC to count all kmers into a database
```
kmer_len=9
threads=8
maxRam=32
minCounts="0"
file_type="fq"
kmcCounterVal="4294967295"

mkdir kmc_dir

for kmer_len in 11 21 31 ; do
echo $kmer_len
for id in SFHH005aa SFHH005ab SFHH005ac SFHH005ad SFHH005ae ; do
echo $id
ls -1 in/${id}_*.fastq.gz > kmc_dir/kmc_input
./bin/kmc -k${kmer_len} -t${threads} -m${maxRam} -cs${kmcCounterVal} -ci${minCounts} -b -${file_type} @kmc_dir/kmc_input kmc_dir/${id}.${kmer_len} ${TMPDIR} 2>> kmc_dir/kmc.err >> kmc_dir/kmc.out
done
done

```




```
for kmer_len in 11 21 31 ; do
echo $kmer_len
for id in SFHH005aa SFHH005ab SFHH005ac SFHH005ad SFHH005ae ; do
id=${id}.${kmer_len}
echo $id
./bin/kmc_tools transform kmc_dir/${id} dump -s kmc_dir/${id}.raw.tsv
cat kmc_dir/${id}.raw.tsv | datamash sum 2 > kmc_dir/${id}.total_kmer_count
total_kmer_count=$( cat kmc_dir/${id}.total_kmer_count )
awk -v sample=${id} -v total_kmer_count=${total_kmer_count} 'BEGIN{FS=OFS="\t";print "kmer",sample}{$2=$2*1000000000/total_kmer_count;print}' kmc_dir/${id}.raw.tsv > kmc_dir/${id}.normalized.tsv
done
done

```



```
for kmer_len in 11 21 31 ; do
echo $kmer_len
i=0
for tsv in kmc_dir/*.${kmer_len}.normalized.tsv ; do
if [ $i -eq 0 ] ; then
base_tsv=${tsv}
else
echo "Joining ${base_tsv} ${tsv}"
join --header -a1 -a2 -e0 -oauto ${base_tsv} ${tsv} > kmc_dir/joined.${kmer_len}.normalized.tmp.${i}.tsv
base_tsv=kmc_dir/joined.${kmer_len}.normalized.tmp.${i}.tsv
fi
i=$[i+1]
done
sed -i -e 's/ /\t/g' ${base_tsv}
done
```







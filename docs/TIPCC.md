
#	TIPCC


```
#	put jobs on hold ( qhold )

qhold 1442460
qhold $( qstat | grep gwendt | awk -F. '{print $1}' | paste -s )
qhold $( qstat | grep gwendt | grep .sr | awk -F. '{print $1}' | paste -s )
qhold $( qstat | grep gwendt | grep .sr | awk -F. '{print $1}' | paste -s )
qhold $( qstat | grep gwendt | grep hrna_1 | awk -F. '{print $1}' | paste -s )
qhold $( qstat | grep gwendt | grep hjd | awk -F. '{print $1}' | paste -s )

qhold $( qstat | grep gwendt | grep .mjf.17 | awk '( $10 == "Q" )' | awk -F. '{print $1}' | paste -s )


#	take jobs off hold ( qrls )

qrls $( qstat | grep gwendt | awk -F. '{print $1}' | paste -s )
qrls $( qstat | grep gwendt | grep hrna_11 | awk -F. '{print $1}' | paste -s )



#	Just Running

qstat | awk '($10 == "R")'


#	Queued or Running 

qstat | awk '($10 ~ /R|Q/)'




checkjob #####

tipcc status --scheduler

echo "echo testing" | qsub -N testing

tipcc node n2
tipcc node n6
tipcc node n38

```


Modules that I load

```BASH
module load CBC
module load htop
module load openssl/1.1.1a
module load python/2.7.15
module load gcc/4.9.2
module load r/3.6.1
module load jdk/8
module load gatk/4.0.2.1
module load coreutils/8.6
module load sqlite
module load cufflinks
module load cmake
module load gawk
module load bedtools2
module load git git-lfs
```




Big items missing from the cluster, all mostly due to the very old OS.

```

Python3

RepeatMasker

Docker

MySQL / MariaDB

Megan and Megan Tools



MetaGO requires dsk 1.6066 (and parse_results) which won't run
./dsk
./dsk: /opt/gcc/gcc-4.9.2/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by ./dsk)
./dsk: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by ./dsk)
./parse_results 
./parse_results: /opt/gcc/gcc-4.9.2/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by ./parse_results)
./parse_results: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by ./parse_results)


from gotcloud
ftp://share.sph.umich.edu/gotcloud/1.16/gotcloud-bin_1.16.tar.gz
[gwendt@cclc01 ~/gotcloud/bin]$ ./vcfCooker 
./vcfCooker: /lib64/libc.so.6: version `GLIBC_2.15' not found (required by ./vcfCooker)
./vcfCooker: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by ./vcfCooker)
```

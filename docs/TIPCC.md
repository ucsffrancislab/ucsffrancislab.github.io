
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


```

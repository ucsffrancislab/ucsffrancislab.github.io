
#	TIPCC


##	Basic Usage

```BASH
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





##	Modules


Modules that I load

```BASH
module load CBC
module load htop
module load openssl/1.1.1a
module load zlib
module load python/2.7.10	#	2.7.10 for gdc-client
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






##	Best Practices


TIPCC has been described as a "fragile flower".
It has many limitations and I've managed to overuse it.

* Don't "stuff" the queue.
* Use fewer, long jobs rather than more, short jobs.
* Use local scratch as much as possible.



##	Local Scratch


Use `/scratch/$USER/job/$PBS_JOBID` in your scripts.

In some of my jobs this seems like a rather extreme response.
Copying input files and references is hundreds of gigabytes.
It would almost seem better to maintain single copies of references on local machines.


Clean up when you're done by adding something like the following to your scripts.

```BASH
trap "{ cd /scratch/; chmod -R +w $SCRATCH_JOB/; \rm -rf $SCRATCH_JOB/ ; }" EXIT
```

I've noticed many times that this doesn't actually work.
My scripts usually include ...

```BASH
set -e	#	exit if any command fails
set -u	#	Error on usage of unset variables
set -o pipefail
```

so I'm not sure exactly what circumstances result in not cleaning up.






##	Array jobs

Array jobs call the same script some number of times and the script must deal with the ARRAYID.
This is most commonly done by parsing a file listing or some file's contents.
The addition of a `%NUMBER` is intended to limit the number of jobs that could run at the same time.
Perhaps this minimizes the load on the scheduler by keeping jobs on hold?
In general, do jobs on hold have less impact on the scheduler?


```BASH
$ jobid=$( echo 'echo ":$PBS_ARRAYID:"; echo ":$PBS_JOBID:"' | qsub -N array -t 1-10%2 -j oe -o array )
$ echo $jobid
1816232[].cclc01.som.ucsf.edu

$ qstat -t ${jobid}
cclc01.som.ucsf.edu: 
                                                                                  Req'd    Req'd       Elap
Job ID                  Username    Queue    Jobname          SessID  NDS   TSK   Memory   Time    S   Time
----------------------- ----------- -------- ---------------- ------ ----- ------ ------ --------- - ---------
1816232[1].cclc01.som.  gwendt      batch    array-1             --      1      1    2gb  99:23:59 Q       -- 
1816232[2].cclc01.som.  gwendt      batch    array-2             --      1      1    2gb  99:23:59 Q       -- 
1816232[3].cclc01.som.  gwendt      batch    array-3             --      1      1    2gb  99:23:59 H       -- 
1816232[4].cclc01.som.  gwendt      batch    array-4             --      1      1    2gb  99:23:59 H       -- 
1816232[5].cclc01.som.  gwendt      batch    array-5             --      1      1    2gb  99:23:59 H       -- 
1816232[6].cclc01.som.  gwendt      batch    array-6             --      1      1    2gb  99:23:59 H       -- 
1816232[7].cclc01.som.  gwendt      batch    array-7             --      1      1    2gb  99:23:59 H       -- 
1816232[8].cclc01.som.  gwendt      batch    array-8             --      1      1    2gb  99:23:59 H       -- 
1816232[9].cclc01.som.  gwendt      batch    array-9             --      1      1    2gb  99:23:59 H       -- 
1816232[10].cclc01.som  gwendt      batch    array-10            --      1      1    2gb  99:23:59 H       -- 


$ cat array-9
:9:
:1816232[9].cclc01.som.ucsf.edu:
```








##	Missing


Big items missing from the cluster, all mostly due to the very old OS.

```BASH

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

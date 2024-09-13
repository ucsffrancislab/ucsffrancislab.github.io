

Parallelize part of a bigger job #72




If I have a big pipeline job running and most of it is multithreaded, but part of the job is a long, slow single threaded process that could be broken up into pieces and run in parallel, Think manually processing every read in a bam file. is there a good way to do this? I'm using scratch otherwise I'd probably just submit another job to the queue. I may end up going this way in the end anyway. 

However I was thinking of either using GNU's Parallel, or running things in the background and then using `wait` or have each sub-job touch a "done" file and wait for all to exist. Parallel would likely offer the most control. Wait would likely be the simplest.

Any thoughts or suggestions?






I've opted to manually parallelize some processing which seems to be working.

```
module load picard

mkdir ${TMPDIR}/split
java -jar $PICARD_HOME/picard.jar SplitSamByNumberOfReads \
	--INPUT MY_BAM_FILE.bam \
	--OUTPUT ${TMPDIR}/split \
	--SPLIT_TO_N_FILES ${SLURM_NTASKS:-8} \
	--CREATE_INDEX true

#	Produces format of ... ${TMPDIR}/split/shard_0001.bam

#	NOTE - can't seem to control the number format from 
#	SplitSamByNumberOfReads( always 4 digits with leading zeros)
#	or seq (no leading zeroes) so use printf to force 4-digit with leading zeroes

for i in $( seq ${SLURM_NTASKS:-8} ) ; do
	i=$( printf "%04d" ${i} )
	PROCESS_SPLIT_BAM_IN_BACKGROUND.bash ${TMPDIR}/split/shard_${i}.bam &
done

echo "Waiting for all jobs to complete"
wait < <(jobs -p)

echo "Merge results"
# Insert merge code
```

Oddly, the `uptime` load is high, but viewing `htop` still shows very low CPU% and MEM%.
```
 11:22:56 up 12 days, 17:04,  1 user,  load average: 62.01, 59.23, 69.00
```


`iostat $( df /scratch/ | tail -1 | cut -d" " -f1 ) 1 10` (last entry) shows something like

```
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           7.51    0.00    1.85    0.00    0.00   90.64

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda7              9.00         0.00       144.00          0        144
```

which I translate to mean not too busy.

This seems to run quite fast (a couple hours) on servers where it's alone (n3 or n5). However on our server (n17) where 8 are running at the same time, the job will take a day or two. 

Obviously, there's a bottleneck somewhere that I can't find but at least its running.





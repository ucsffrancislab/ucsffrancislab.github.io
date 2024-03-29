

#	C4 Sample Array script


Essentially, an array script in slurm is a script that is written to use the environment variable `SLURM_ARRAY_TASK_ID`.
This variable is just an integer.
I use it to select a line from a text file or `ls` output.
Nothing really special other than that.



DO NOT `qdel 733558_26`.
I think that the script parses off the first numeric part and cancels the entire array job!

```
qdel 733795_2
Argument "733795_2" isn't numeric in subroutine entry at /usr/bin/qdel line 96, <DATA> line 602.
```

I think that `scancel 733558_26` is OK.





```
#!/usr/bin/env bash
#SBATCH --export=NONE   # required when using 'module'

hostname
echo "Slurm job id:${SLURM_JOBID}:"
date

#set -e  #       exit if any command fails
set -u  #       Error on usage of unset variables
set -o pipefail
if [ -n "$( declare -F module )" ] ; then
	echo "Loading required modules"
	module load CBI samtools/1.13 bowtie2/2.4.4 picard
	#bedtools2/2.30.0
fi
#set -x  #       print expanded command before executing it


OUT="/francislab/data1/working/20220610-EV/20220624-preprocessing_with_custom_umi_sequence_extraction/out"

#while [ $# -gt 0 ] ; do
#	case $1 in
#		-d|--dir)
#			shift; OUT=$1; shift;;
#		*)
#			echo "Unknown params :${1}:"; exit ;;
#	esac
#done

mkdir -p ${OUT}


line=${SLURM_ARRAY_TASK_ID:-1}
echo "Running line :${line}:"


#	Use a 1 based index since there is no line 0.

#r1=$( ls -1 /francislab/data1/raw/20220610-EV/SF*R1_001.fastq.gz | sed -n "$line"p )
sample=$( sed -n "$line"p /francislab/data1/working/20220610-EV/20220624-preprocessing_with_custom_umi_sequence_extraction/metadata.csv | awk -F, '{print $1}' )
r1=$( ls /francislab/data1/raw/20220610-EV/${sample}_*R1_001.fastq.gz )

#	Make sure that r1 is unique. NEEDS the UNDERSCORE AFTER SAMPLE!


echo $r1

if [ -z "${r1}" ] ; then
	echo "No line at :${line}:"
	exit
fi
```




It is called like the following command, where the `--array` option is used to set the range of values to set the environment variable to.
It can be a range like `10-100`, a list like `5,8,12,99`, or just a single number `13`. 
It is followed by the number of jobs you'd like it to try to run at the same time like `%1` or `%8`.
Setting it to 0 means it will try to run as many as it can, I think.


```
sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL --array=1-86%1 --job-name="preproc" --output="/francislab/data1/working/20220610-EV/20220624-preprocessing_with_custom_umi_sequence_extraction/logs/preprocess.${date}-%A_%a.out" --time=2880 --nodes=1 --ntasks=8 --mem=60G --gres=scratch:250G /francislab/data1/working/20220610-EV/20220624-preprocessing_with_custom_umi_sequence_extraction/array_wrapper.bash
```


You can modify the array job handler, but some things are restricted.
If you'd like to throttle up or down the number of jobs running you can use a command like 

```
scontrol update ArrayTaskThrottle=6 JobId=352083
```





##	Single self-calling example script


```
#!/usr/bin/env bash
#SBATCH --export=NONE   # required when using 'module'

hostname
echo "Slurm job id:${SLURM_JOBID}:"
date

#		Will be "slurm_script" at runtime
if [ -n "${SLURM_JOB_NAME}" ] ; then
	script=${SLURM_JOB_NAME}
else
	script=$( basename $0 )
fi

#	PWD preserved by slurm for where job is run? I guess so.
arguments_file=${PWD}/${script}.arguments

if [ -n "${SLURM_ARRAY_TASK_ID}" ] ; then

	set -e	#	exit if any command fails
	set -u	#	Error on usage of unset variables
	set -o pipefail
	if [ -n "$( declare -F module )" ] ; then
		echo "Loading required modules"
		module load CBI bcftools/1.15.1
	fi
	set -x	#	print expanded command before executing it

	line=${SLURM_ARRAY_TASK_ID:-1}
	echo "Running line :${line}:"

	#	Use a 1 based index since there is no line 0.

	args=$( sed -n "$line"p ${arguments_file} )
	echo $args

	if [ -z "${args}" ] ; then
		echo "No line at :${line}:"
		exit
	fi

	#	Do what you gotta do

	echo "Complete"

else

	date=$( date "+%Y%m%d%H%M%S" )

	cat <<- EOF > ${arguments_file}
SFHH011AC
SFHH011BB
SFHH011BZ
SFHH011CH
SFHH011I
SFHH011S
SFHH011Z
SFHH011BO
	EOF

	#	or some file specific list
	#	ls -1 in/*bam | xargs -I% basename % > ${arguments_file}

	max=$( cat ${arguments_file} | wc -l )

	mkdir -p ${PWD}/logs/
	date=$( date "+%Y%m%d%H%M%S%N" )
	sbatch --mail-user=$(tail -1 ~/.forward)  --mail-type=FAIL \
		--array=1-${max}%16 --job-name=${script} \
		--output="${PWD}/logs/${script}.${date}.%A_%a.out" \
		--time=1440 --nodes=1 --ntasks=4 --mem=30G \
		$( realpath ${0} )

fi
```

There are many ways to do this and I have expanded on this since its writing.









##	Too large


There are limits on how many jobs can be in your array job.
Not sure why, but there is.
Perhaps there is an additional limit on the number of total jobs that can be submitted.
If you cross that line, you'll see something like ...

```
sbatch: error: QOSMaxSubmitJobPerUserLimit
sbatch: error: Batch job submission failed: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)
```


If this is expected to be common, one could create a single job with ample resources and run all of their "jobs" with parallel.
This clearly won't be as flexible once started as it is effectively just 1 slurm job.

Something like ...

```
echo Command1 > commands.txt
echo Command2 >> commands.txt
echo Command3 >> commands.txt
#etc.

parallel -j ${SLURM_NTASKS:-32} < commands.txt

```

These are 2 completely different job submission types so you'd need to write 2 different scripts.








#	C4 Sample Array script


Essentially, an array script in slurm is a script that is written to use the environment variable `SLURM_ARRAY_TASK_ID`.
This variable is just an integer.
I use it to select a line from a text file or `ls` output.
Nothing really special other than that.



DO NOT `qdel 733558_26`.
I think that the script parses off the first numeric part and cancels the entire array job!
`qdel 733795_2`
Argument "733795_2" isn't numeric in subroutine entry at /usr/bin/qdel line 96, <DATA> line 602.

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




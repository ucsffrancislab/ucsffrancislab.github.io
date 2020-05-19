
#	Reference Masking


Using Repeat Masker to block non-specific regions from a reference to minimize false identification.


Aligning raw reads to a viral reference can result in small regions with extremely deep coverage and no coverage anywhere else.

We were find a BeAnn virus quite often.

Many times simply running RepeatMasker on the viral fasta before creating the reference does a good job.

However, sometimes even that isn't good enough.

We've gone so far as to chop up the viral fasta and align it to a human reference and then mask those regions that aligned as well.



Setup

```BASH
for f in NC_00????.?.masked.fasta ; do
for s in 50 25 ; do

	echo $f

	b=$( basename $f .fasta )
```

	Split the fasta file into pieces.
	We chose 50bp with an extra 50bp overlap into a single file.
	And renamed the output file.

```BASH
	faSplit -oneFile -extra=${s} size ${f} ${s} split
	mv split.fa ${b}.split.${s}.fa
```


	In this particular case, we needed a human reference that did not include EBV.
	Aligning locally performed best.
	We attempted a number of settings but stuck with --very-sensitive-local



```BASH
	bowtie2 -x hg38-noEBV -f -U ${b}.split.${s}.fa --very-sensitive-local --no-unal -S ${b}.split.${s}.loc.sam
```

Above and below NEED the complete reference (with its version number as it is in the fasta)

The sequence names are sequential so we can calculate the positions based on this number, the size and the overlap.
We do this in the first awk command.

The second awk command attempts to expand and merge these regions.


```BASH

	echo make bed
	samtools view ${b}.split.${s}.loc.sam | awk -v s=${s} -v ref=${b%.masked} '{
		sub(/^split/,"",$1);
		a=1+s*$1
		b=a+(2*s-1)
		print ref"\t"a"\t"b
	}' | awk 'BEGIN{FS=OFS="\t"}
	(NR==1){
		r=$1
		if($2>50){
			s=$2-50
		}else{
			s=1
		}
		e=$3+50
	}
	(NR!=1){
		if( $2 <= (e+50+1) ){
			e=$3+50
		}else{
			print $1,s,e
			s=$2-50
			e=$3+50
		}
	}
	END{
		print r,s,e
	}' > ${b}.${s}.loc.mask.bed
```

Finally, masking these regions and creating a new fasta using a `bedtool`

```BASH
	echo maskFastaFromBed
	maskFastaFromBed -fi ${f} -fo ${b}.${s}.loc-masked.fa -bed ${b}.${s}.loc.mask.bed -fullHeader

done
done
```


Sadly, I still find a few regions of extreme depth.
This requires further investigation.




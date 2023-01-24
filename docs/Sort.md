
#	Sort

##	Benchmarking sorting a BAM file.


Searching for the fastest sorting


```
time bamsort sortthreads=4 I=HT-7604-01A-11D-2088.bam O=HT-7604-01A-11D-2088.bamsort.bam

real	15m32.057s
user	15m56.516s
sys	0m16.190s

real	15m15.331s
user	15m40.937s
sys	0m15.542s

real	16m6.561s
user	16m20.839s
sys	0m19.519s



time bamsort inputthreads=4 sortthreads=4 I=HT-7604-01A-11D-2088.bam O=HT-7604-01A-11D-2088.bamsort.bam

real	14m35.514s
user	16m23.456s
sys	0m17.569s




time bamsort inputthreads=4 sortthreads=4 outputthreads=4 I=HT-7604-01A-11D-2088.bam O=HT-7604-01A-11D-2088.bamsort.bam

real	5m28.488s
user	18m10.684s
sys	0m22.078s

```






```
time samtools sort -@ 3 -o HT-7604-01A-11D-2088.samtools.bam HT-7604-01A-11D-2088.bam
   
real	6m43.247s
user	22m4.258s
sys	0m29.035s

real	6m38.341s
user	21m51.209s
sys	0m27.287s

real	6m21.743s
user	20m59.858s
sys	0m27.886s

real	6m48.929s
user	22m11.218s
sys	0m27.769s
```






```
time sambamba sort -t 4 -o HT-7604-01A-11D-2088.sambamba.bam HT-7604-01A-11D-2088.bam

sambamba 1.0.0
 by Artem Tarasov and Pjotr Prins (C) 2012-2022
    LDC 1.30.0 / DMD v2.100.1 / LLVM14.0.6 / bootstrap LDC - the LLVM D compiler (1.28.1)

real	5m5.473s
user	22m22.874s
sys	0m17.537s

real	5m13.226s
user	22m51.997s
sys	0m19.026s

real	4m49.165s
user	21m16.796s
sys	0m15.319s

real	4m52.493s
user	21m31.413s
sys	0m15.984s
```




biobambam2 bamsort seems to be a bit faster, but this wasn't the most thorough comparison.


```
date; time bamsort inputthreads=4 sortthreads=4 outputthreads=4 I=HT-7604-01A-11D-2088.name.bam O=HT-7604-01A-11D-2088.bamsort.bam; date
Tue Jan 24 09:25:17 MST 2023

real	5m41.268s
user	19m17.392s
sys	0m32.100s



date ; time samtools sort -@ 3 -o HT-7604-01A-11D-2088.samtools.bam HT-7604-01A-11D-2088.name.bam ; date

real	6m56.791s
user	22m50.040s
sys	0m26.584s



date ; time sambamba sort -t 4 -o HT-7604-01A-11D-2088.sambamba.bam HT-7604-01A-11D-2088.name.bam ; date

real	5m34.947s
user	23m24.191s
sys	0m17.613s
```










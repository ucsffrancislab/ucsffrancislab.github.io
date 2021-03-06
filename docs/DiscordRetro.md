
#	DiscordRetro


<https://github.com/adamewing/discord-retro>

Find non-reference TE insertions from discordant read pair mappings


```

discord-retro is a tool to identify transpoable element (TE) insertions from
paired-end whole-genome sequence data, specifically tuned for Illumina reads.

There are a number of auxilliary files that specify details of the various
TEs one expects to find. Currently, discord-retro is configured to analyse
human genomes; this can be changed by substituting the appropriate annotations
for another species of interest.

More details to come, but for now, users can try running discord-retro on the
example .bam included (test/example.bam) with the following commandline:

./discord-retro.py -s samplelist_example.txt -c human_sample.cfg -o test/output

in order for this to work, human_sample.cfg will have to be modified to assign
'hg19' to a copy of hg19 that has been indexed by `bwa index -a bwasw`

output will end up in test/output/example, the most relevant files are
other.tab.txt and otherbreaks.tab.txt, the former contains the coordinates and
annotations, the latter contains breakpoint information.

Prerequisites:

bwa (http://bio-bwa.sourceforge.net/)
samtools (http://samtools.sourceforge.net/)
tabix (http://samtools.sourceforge.net/tabix.shtml)
pysam (http://code.google.com/p/pysam/)
parallelpython (http://www.parallelpython.com/)
```

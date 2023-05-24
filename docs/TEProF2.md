
#	TEProF2



https://github.com/twlab/TEProf2Paper


/francislab/data1/refs/sources/gencodegenes.org/

/francislab/data1/refs/sources/genome.ucsc.edu/



This analysis seems to be strand specific.

Determine it using ... [RNAStrandedness](docs/RNAStrandedness)


```

TEProF2_array_wrapper.bash --threads 4 --strand --rf \
  --in  IN_PATH_CONTAINING_ALIGNED_BAM_FILES \
  --out OUT_PATH_TO_CONTAIN_INDIVIDUALLY_PREPROCESSED_FILES \
  --extension .Aligned.sortedByCoord.out.bam

```


TCGA33 Guided

```

TEProF2_aggregation_steps.bash --threads 64 --strand --rf \
  --reference_merged_candidates_gtf /francislab/data1/refs/TEProf2/reference_merged_candidates.gtf \
  --in  IN_PATH_TO_CONTAIN_INDIVIDUALLY_PREPROCESSED_FILES \
  --out OUT_PATH_TO_CONTAIN_AGGREGATED_PROCESSED_FILES

```




Extract the TPM Expression Table from the RData.

```
module load r
R

load("out/Step13.RData")

tpmexpressiontable[0:5,0:5]
  TranscriptID TCONS_00000050 TCONS_00000056 TCONS_00000058 TCONS_00000059
2       260v01    0.009950144    0.018558406      0.0000000      0.4937565
3       260v02    0.075301157    0.000000000      0.5414942      0.0000000
4       260v03    0.486996922    0.037251096      0.3591145      0.0000000
5       260v04    0.625168363    0.000000000      1.6914543      0.0000000
6       260v05    2.446610945    0.003706618      0.2736372      0.2831258


row.names(tpmexpressiontable)=tpmexpressiontable[['TranscriptID']]
df = subset(tpmexpressiontable, select = -c(TranscriptID) )

df[0:5,0:5]
       TCONS_00000050 TCONS_00000056 TCONS_00000058 TCONS_00000059
260v01    0.009950144    0.018558406      0.0000000      0.4937565
260v02    0.075301157    0.000000000      0.5414942      0.0000000
260v03    0.486996922    0.037251096      0.3591145      0.0000000
260v04    0.625168363    0.000000000      1.6914543      0.0000000
260v05    2.446610945    0.003706618      0.2736372      0.2831258

write.csv(df,file='out/tpmexpressiontable.csv', quote=FALSE)
write.csv(t(df),file='out/tpmexpressiontable.t.csv', quote=FALSE)

```







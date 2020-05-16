
Let's evaluate the expression of each sample to LTRs and/or SVAs.

We could find/create a reference and directly align our samples to this reference and count the reads that successfully align.

Or we could align to a human reference and then use featureCounts from [subread](http://subread.sourceforge.net/) and a feature file.

While I can't find a gtf/gff file for LTRs and SVAs, there are bed files available through IGV.


```
IGV
File -> Load from Server ...
Available Datasets -> Annotations -> Repeat Masker 
LTR
Retroposon
```


Once loaded, hovering over the gene track will show a URL to download the bed files.

Or we could look directly in the [IGV github repo](https://github.com/igvteam/igv/blob/master/genomes/db/hg38/hg38_annotations.xml)


```BASH
wget https://s3.amazonaws.com/igv.org.genomes/hg38/rmsk/hg38_rmsk_LTR.bed.gz

zcat hg38_rmsk_LTR.bed.gz | head
chr1	21948	22075	MLT1K	254	+
chr1	30693	30848	MLT1A	741	+
chr1	30952	31131	MLT1A	741	+
chr1	34564	34921	MLT1J2	850	-
chr1	38255	39464	MLT1E1A-int	3877	+
chr1	39464	39623	MLT1E1A	750	+
chr1	39924	40294	MLT1E1A	783	+
chr1	40735	40878	LTR16C	260	-
chr1	41393	42273	ERV3-16A3_I-int	765	-
chr1	42369	42504	MamRep1527	341	+

zcat hg38_rmsk_LTR.bed.gz | wc -l
766019
```


We can see that the 4th column (name) is repeated. How often?


```BASH
zcat hg38_rmsk_LTR.bed.gz | awk '{print $4}' | sort | uniq -c | wc -l
595
```


Very often. This number is very close to that referenced in the https://www.nature.com/articles/s41467-019-13035-2 data.

The `bedtools` that can be used to convert a bed file to a gtf will make an ugly gtf and treats each line separately and I've found no option to merge them.


```BASH
bedToGenePred hg38_rmsk_LTR.bed.gz stdout | genePredToGtf file stdin stdout | head
chr1	stdin	exon	21949	22075	.	+	.	gene_id "MLT1K"; transcript_id "MLT1K"; exon_number "1"; exon_id "MLT1K.1";
chr1	stdin	CDS	21949	22075	.	+	0	gene_id "MLT1K"; transcript_id "MLT1K"; exon_number "1"; exon_id "MLT1K.1";
chr1	stdin	start_codon	21949	21951	.	+	0	gene_id "MLT1K"; transcript_id "MLT1K"; exon_number "1"; exon_id "MLT1K.1";
chr1	stdin	exon	30694	30848	.	+	.	gene_id "MLT1A"; transcript_id "MLT1A"; exon_number "1"; exon_id "MLT1A.1";
chr1	stdin	CDS	30694	30848	.	+	0	gene_id "MLT1A"; transcript_id "MLT1A"; exon_number "1"; exon_id "MLT1A.1";
chr1	stdin	start_codon	30694	30696	.	+	0	gene_id "MLT1A"; transcript_id "MLT1A"; exon_number "1"; exon_id "MLT1A.1";
chr1	stdin	exon	30953	31131	.	+	.	gene_id "MLT1A_2"; transcript_id "MLT1A_2"; exon_number "1"; exon_id "MLT1A_2.1";
chr1	stdin	CDS	30953	31131	.	+	0	gene_id "MLT1A_2"; transcript_id "MLT1A_2"; exon_number "1"; exon_id "MLT1A_2.1";
chr1	stdin	start_codon	30953	30955	.	+	0	gene_id "MLT1A_2"; transcript_id "MLT1A_2"; exon_number "1"; exon_id "MLT1A_2.1";
chr1	stdin	exon	34565	34921	.	-	.	gene_id "MLT1J2"; transcript_id "MLT1J2"; exon_number "1"; exon_id "MLT1J2.1";
```


We can see that each name is repeated often.


```BASH
zcat hg38_rmsk_LTR.bed.gz | awk '{print $4}' | sort | uniq -c | head
    357 ERV24B_Prim-int
    430 ERV24_Prim-int
   6681 ERV3-16A3_I-int
   1115 ERV3-16A3_LTR
   4268 ERVL-B4-int
   8778 ERVL-E-int
    884 ERVL-int
    215 ERVL47-int
      2 EUTREP15
      2 EUTREP16
```


Not certain why 1 is added to the start position, but bedtools did it so monkey see, monkey do.
I'll keep things simple. We don't know all of the other tags and certainly don't need multiple entries for each.


```BASH
bedToGenePred hg38_rmsk_LTR.bed.gz stdout | awk 'BEGIN{FS=OFS="\t"}{print $2,"source","feature",$4+1,$5,".",$3,".","feature_name \""$1"\""}' > hg38_rmsk_LTR.jake.gtf

head hg38_rmsk_LTR.jake.gtf
chr1	source	feature	21949	22075	.	+	.	feature_name "MLT1K"
chr1	source	feature	30694	30848	.	+	.	feature_name "MLT1A"
chr1	source	feature	30953	31131	.	+	.	feature_name "MLT1A"
chr1	source	feature	34565	34921	.	-	.	feature_name "MLT1J2"
chr1	source	feature	38256	39464	.	+	.	feature_name "MLT1E1A-int"
chr1	source	feature	39465	39623	.	+	.	feature_name "MLT1E1A"
chr1	source	feature	39925	40294	.	+	.	feature_name "MLT1E1A"
chr1	source	feature	40736	40878	.	-	.	feature_name "LTR16C"
chr1	source	feature	41394	42273	.	-	.	feature_name "ERV3-16A3_I-int"
chr1	source	feature	42370	42504	.	+	.	feature_name "MamRep1527"
```


`featureCounts` will treat each of these seperately, but in the final csv it will concatenate the chromosome, start, finish and strand fields for each matching label, and it seems to total up the length.

This is irrelevant, as we will remove these fields anyway with a command like ... `sed -r 's/\t\S+\t\S+\t\S+\t\S+\t\S+//1' LTR_features.csv > LTR_features.new.csv`

Let's do the same for the SVAs.


```BASH
wget https://s3.amazonaws.com/igv.org.genomes/hg38/rmsk/hg38_rmsk_Retroposon.bed.gz

zcat hg38_rmsk_Retroposon.bed.gz | head
chr1	837098	837180	SVA_D	276	+
chr1	6304393	6306236	SVA_D	9901	-
chr1	6714507	6716330	SVA_D	6435	-
chr1	7600485	7602002	SVA_D	11216	-
chr1	7952097	7953558	SVA_D	5411	+
chr1	8079151	8079209	SVA_F	308	-
chr1	8260046	8260103	SVA_B	251	+
chr1	8260237	8260251	SVA_B	251	+
chr1	8500394	8501159	SVA_D	4299	+
chr1	8501169	8503087	SVA_E	6406	+

zcat hg38_rmsk_Retroposon.bed.gz | wc -l
5882

zcat hg38_rmsk_Retroposon.bed.gz | awk '{print $4}' | sort | uniq -c | wc -l
6

zcat hg38_rmsk_Retroposon.bed.gz | awk '{print $4}' | sort | uniq -c
   1147 SVA_A
    873 SVA_B
    527 SVA_C
   1583 SVA_D
    709 SVA_E
   1043 SVA_F

bedToGenePred hg38_rmsk_Retroposon.bed.gz stdout | awk 'BEGIN{FS=OFS="\t"}{print $2,"source","feature",$4+1,$5,".",$3,".","feature_name \""$1"\""}' > hg38_rmsk_Retroposon.jake.gtf
```


And concatenate them.


```BASH
cat hg38_rmsk_LTR.jake.gtf hg38_rmsk_Retroposon.jake.gtf > hg38_rmsk_LTR_Retroposon.jake.gtf
```



```BASH
~/.local/bin/featureCounts.bash \
	-T 16 -a hg38_rmsk_LTR_Retroposon.jake.gtf \
	-t feature -g feature_name \
	-o LTR_SVA_features.csv *.STAR.hg38.Aligned.out.bam"

tail -n +2 LTR_SVA_features.csv | sed -r 's/\t\S+\t\S+\t\S+\t\S+\t\S+//1' > LTR_SVA_features.new.csv

tail -n +2 LTR_SVA_features.new.csv | head -3
Geneid	Length	s001	s002	s003	s004	s005	s006	s007	s008	s009	s010	s012	s013	s014	s015	s017	s018	s019	s020	s021	s023	s024	s025	s026	s028	s029	s033	s034	s036	s037	s038	s040	s041	s043	s044	s045	s046	s047	s048	s049	s051	s052	s053	s054	s055	s056	s057	s058	s059	s060	s061	s062	s063	s064	s065	s067	s068	s069	s070	s071	s072	s073	s074	s075	s076	s077	s079	s080	s083	s084	s086	s087	s089	s090	s091	s092	s093	s094	s095	s096	s097	s098	s099	s100	s101	s102	s103	s104	s105	s106	s107	s108	s110	s111	s112	s113	s114	s115	s116	s117	s119	s120	s122	s123	s124	s125	s126	s127	s128	s129	s130	s132	s133	s134	s135	s137	s139	s141	s142	s143	s144	s145	s146	s147	s150	s151	s152	s153	s155	s156	s157	s158	s159	s160	s161	s162	s163	s164	s165	s166	s167	s168	s169	s170	s171	s173	s174	s175	s176	s178	s179	s180	s181	s182	s183	s184	s185	s186	s187	s188	s189	s190	s191	s192	s193	s194	s197	s198	s199	s200	s201	s202	s203	s204	s205	s206	s207	s208	s209	s210	s212	s213	s214	s215	s216	s217	s218	s219	s221	s222	s224	s225	s226	s227	s228	s230	s231	s232	s233	s234	s235
MLT1K	4019453	9640	9010	3696	7187	10878	9041	7289	8477	5774	13994	5816	7426	7658	4594	8486	8255	7251	7668	9505	5314	7398	12797	22960	21627	10011	14769	12159	10297	20500	14047	13519	9024	6693	12407	10455	7892	10178	6636	6701	7779	9792	4663	14697	7834	14427	18964	8924	11214	13611	10872	13581	12352	10580	10295	10975	9656	12285	11883	9449	12038	5887	9650	12081	12676	13176	7519	12732	14170	17	6865	11296	10603	7306	4324	6502	9867	10886	11110	11116	7892	11121	8497	6385	10998	15508	10145	10809	6531	7929	7803	9626	6676	13075	10131	12508	7071	12362	4103	3157	19452	10130	11369	10013	9630	11451	11249	12194	12621	15438	12420	12489	9246	8122	11109	11584	9588	10635	13896	10030	10813	9526	7544	5286	11396	14107	11565	10823	9976	10594	7510	9911	8115	10597	10700	11082	10239	7615	14911	12322	11079	13173	7989	11624	6317	6997	8523	6162	9003	8064	5585	9527	10777	6695	8448	7201	5105	9771	8022	7298	6943	8037	9153	5036	8079	7981	8515	8149	8520	7842	7281	7422	6639	15129	7131	7581	8095	7290	5797	6955	7210	6247	7391	9310	6175	7719	5774	7071	9120	8075	6132	5989	4423	12107	5753	5789	8439	6980	7364	8890	7691
MLT1A	2578604	4394	7521	1985	4716	5274	3781	5315	5127	2753	7936	3169	4135	2901	2435	3532	4421	3202	2613	4279	2896	3149	7146	9837	9436	4684	8012	5858	6137	9765	8081	7053	5043	3153	5615	4269	3122	4433	3324	3597	3952	3957	1961	4747	4039	6844	11073	4802	5579	7442	6382	8316	4318	5566	4621	5558	3878	5377	5098	4993	5665	2857	5171	4626	5497	6030	3975	6740	5815	4	2837	4890	4406	3631	2438	4473	4213	3073	7327	6098	3932	5841	4333	2873	6365	7272	4325	6434	3266	3838	3132	5314	3531	5123	5003	5673	2771	5554	3286	4190	10526	4441	4965	3891	5936	5475	6649	6037	5654	7106	6082	4670	4309	3291	4441	6176	4169	3831	7897	4413	6552	3889	3755	2256	4813	7413	4937	4373	5752	4846	3091	4083	3929	5319	5127	6424	3985	3453	6443	6046	5017	6349	3216	4327	2740	3442	3679	2975	4413	3815	2039	4493	5107	3049	3854	3284	3146	4055	3907	3579	2856	4160	4288	2016	3870	3894	3331	3944	3863	3339	3654	2955	2963	8319	3015	3160	4382	3879	2977	4190	2919	2882	2806	4630	2170	3638	2255	3150	3814	3086	3373	3257	2335	4850	2561	2592	3895	3032	2919	3504	3597
```









#	Pipeline Notes


##	SNP2HLA 

https://github.com/ucsffrancislab/genomics/tree/master/data/20200724-UKB/20200804-SNP2HLA 

##	Imputation 

https://github.com/ucsffrancislab/genomics/tree/master/data/20200520_Adult_Glioma_Study_GWAS/20200520_prep_for_imputation 

##	Preprocess 

###	Bbmap / bbduk  

Trim adapters and low base quality 

Then filter out read pairs that are no longer the same length 

Investigate discontinuity in read length distribution? 

###	Cutadapt 
 

##	Any 

###	K-mer Analysis 

####	IMOKA 

Proving very useful 

####	GECKO 

Way too long and a bit confusing 

####	MetaGO 

Select (grep?) reads and mate with k-mer (and reverse complement) 

Count? 

Align human? 

Align anything? 

Try to assemble with velvet or ABYSS? 

####	HAWK  

takes way too long 

###	Catchitt 

Transcription Factor Binding Prediction 

https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1614-y 

Requires ChIP-seq, DNase-seq and ATAC-seq data 

https://github.com/kundajelab/atac_dnase_pipelines 

https://github.com/ENCODE-DCC/atac-seq-pipeline 

http://jstacs.de/index.php/Catchitt#Tutorial_using_ENCODE_data 

https://github.com/ENCODE-DCC/chip-seq-pipeline 

 

##	RNA / Exome 

####	REdiscoverTE (from FASTQ)  

####	STAR Align hg38 

FeatureCount 

DESeq? ( DXSeq? ) 

####	Kallisto / Sleuth 

####	CIRCexplorer2 

https://genome.cshlp.org/content/26/9/1277.full 

https://www.picb.ac.cn/rnomics/circpedia/ 

https://circexplorer2.readthedocs.io/en/latest/ 

 

##	DNA 

####	HKLE Chimera (from FASTQ) 

Local align to HKLEs so can soft clip 

Select and trim those that aligned at and passed an end 

Align to human to find insertion points 

####	Metagenomic Classification 

Kraken2 / ABV – Bracken 

DESeq? 

####	Human alignment 

Dark 

Unmapped alignment with blast or diamond to nt or nr 

Consensus VCF 

Tumor / Normal (from aligned BAMs) 

Manta 

Strelka 

Mutation Signature Analysis 




Reference similar pipelines in https://docs.gdc.cancer.gov/Data/Introduction/ 



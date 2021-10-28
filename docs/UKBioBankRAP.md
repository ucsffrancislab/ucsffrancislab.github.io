#	UKBioBankRAP

##	Usage


###	Tutorials

https://dnanexus.gitbook.io/uk-biobank-rap/getting-started/platform-overview




##	How much?


General access to sequencing data (WGS and WES) is 9,000 £ for 3 years. 1,000 £ for each additional institution if we are sharing access.


Storage costs on S3 are inline with AWS S3 storage costs ( 0.0141 £ per GB per month ). 


Downloading any produced data is 0.0396 £ per GB. I don't think that anyone is permitted to download the raw data. I suppose your pipeline could do nothing producing a duplicate file that you could download if you really wanted to.


Processing costs are also incline with AWS EC2 pricing. For example, an large instance with similar resources as our server, r5d.16xlarge has 64 CPU, 512GB RAM and 2.4 TB SSD storage costs 2.336 £ per hour.


https://www.ukbiobank.ac.uk/enable-your-research/costs

https://dnanexus.gitbook.io/uk-biobank-rap/working-on-the-research-analysis-platform/billing-and-costs

https://dnanexus-prod-asg-dnanexusprodassets4d7ed69b-i607e894f3ya.s3.us-east-1.amazonaws.com/images/files/UKB_Rate_Card-Current.pdf


##	Can data request be expanded to include more?




##	Is use project-specific or do users have "free reign" to run alternate analyses once access is granted?








#	Cancer Genomics Cloud (CGC)



https://www.cancergenomicscloud.org/


https://cgc.sbgenomics.com/



https://docs.cancergenomicscloud.org/




##	Access

Uses [eRA Commons](docs/eRACommons) credentials







##	Create Tools



###	Create Docker Image


Open one, edit, save it. Use a Dockerfile. Whatever. Just have an image with all your apps in it.

```
docker build -t melt --file MELT.Dockerfile ./
```



###	Authentication Token

You're gonna need a token (password) to upload.


https://docs.sevenbridges.com/docs/get-your-authentication-token


Visit : https://cgc.sbgenomics.com/developer/token

Click ... Developer > Authentication Token

Click "Generate Token" (Only valid for about 4 months)

Copy and keep it somewhere secure?

It will be available at the website, so you can return to get it when needed.



###	Login


```
docker login --username wendt2017 cgc-images.sbgenomics.com
```

Use your token as the password



###	Upload Docker Image


https://docs.sevenbridges.com/docs/upload-your-docker-image-1


Tag it to mesh with CGC's rules

```
docker tag melt cgc-images.sbgenomics.com/wendt2017/melt:20230116
```


Upload it

```
docker push cgc-images.sbgenomics.com/wendt2017/melt:20230116
```


You can find it in ...

Click ... Developer > Docker registry to see your images




###	New Tools



Create a new tool using your Docker image


Can be very tricky



Expression editor. Language? JSON?
```
$(

)

or

${

	return value
}
```


DOES NOT PRESERVE TEST VALUES WHICH KINDA SUCKS.



Redirect stdout to job.out.log and make that an output port.
I tried this with Preprocess.out. Not sure what happened. Case? .log? Now it does?

java outputs to stderr

java -jar outputs to stdout which is not captured?













##	Python API File Uploading

```
python3 -m pip install --upgrade --user sevenbridges-python
```


Create a credentials file (`$HOME/.sevenbridges/credentials`) with at least the cgc section.

```
[default]
api_endpoint = https://api.sbgenomics.com/v2
auth_token = <TOKEN_HERE>

[cgc]
api_endpoint = https://cgc-api.sbgenomics.com/v2
auth_token = <TOKEN_HERE>


#The configuration file, $HOME/.sevenbridges/credentials, has a simple .ini file format, with the environment (the Seven Bridges Platform, or the CGC, or Cavatica) indicated in square brackets, as shown:
#
#	[default]
#	api_endpoint = https://api.sbgenomics.com/v2
#	auth_token = <TOKEN_HERE>
#
#	[cgc]
#	api_endpoint = https://cgc-api.sbgenomics.com/v2
#	auth_token = <TOKEN_HERE>
#
#	[cavatica]
#	api_endpoint = https://cavatica-api.sbgenomics.com/v2
#	auth_token = <TOKEN_HERE>
#	c = sbg.Config(profile='cgc')
#api = sbg.Api(config=c)
```




```
#!/usr/bin/env python

import sevenbridges as sbg
c = sbg.Config(profile='cgc')
api = sbg.Api(config=c)

api.users.me()

for project in api.projects.query().all():
	print (project.id,project.name)

project = api.projects.get(id='wendt2017/sgdp')

for file in api.files.query(project=project).all():
	print(file.id,file.name)

refs=api.files.query(project=project,names=['references'],limit=1)[0]
bwa=api.files.query(parent=refs.id,names=['bwa'],limit=1)[0]

upload=api.files.upload('hg38.chrXYM_alts.tar', parent=bwa, wait=False)

upload.status
#'PREPARING'

upload.start() # Starts the upload.

upload.status
#'RUNNING'

upload.status

upload.progress

upload.progress

upload.progress

upload.progress

upload.progress

upload.progress
#	0.01000673635349509		#  MAX? I expected this to go to 100 or 1.00

upload.status
#'COMPLETED'

#	Surprisingly fast for 5GB
```




##	Instance types


General Purpose usually have 4:1 ratio of memory to CPU. (m5.2xlarge	8	32)

Compute optimized usually have 2:1 ratio (c5.2xlarge	8	16)

Memory optimized usually have 8:1 ratio (r5.2xlarge	8	64)

Choose appropriately.


| Instance Size | vCPU | Memory (GiB) | Cost per hour (nonSPOT) |
| - | - | - | - |
| m5.2xlarge | 8 | 32 |
| m5.4xlarge | 16 | 64 |
| m5.8xlarge | 32 | 128 | $1.536 |
| m5.12xlarge | 48 | 192 |
| m5.16xlarge | 64 | 256 |
| m5.24xlarge | 96 | 384 |
| r5.4xlarge | 16 | 128 | $1.008 |
| c5.large | 2 | 4 | $0.085 |
| c5.18xlarge | 72 | 144 | $3.06 |
| c5.24xlarge | 96 | 192 | $4.08 |

Not sure if all AWS instance types are available on CGC





##	Pipeline testing


Several times tried SPOT instances which got killed. Irritating. Worth it?

Do failed SPOT instances restart as another SPOT instance?
Can this SPOT instance be canceled just like the first?
Or a standard instance?



Use preemptible instances - By default, instances use on-demand pricing. However, they can also be configured to use “spot” instances. These are preemptible based on bid price and demand. Their pricing does fluctuate, however, it is typically much cheaper than on-demand pricing. There is very little downside to using them on the CGC, as there is a built-in retry mechanism. If your job or a portion of your workflow) is interrupted, it will be retried at on-demand pricing.








Example: You have 5 TB of derived data stored in a project with the location set to AWS-us-east-1. Seven Bridges data incurs $0.021 per GB per month in storage costs for the AWS-us-east-1 cloud location. Therefore, you would expect 5000 GB x $0.021 = $105 to be your monthly storage price.





###	BioBamBam2 BamToFastq



GZIP Output : 1



```
LP6005441-DNA_D11.srt.aln.bam - 75.4 GB

c4.2xlarge [spot] (8cpu/16GB) - excessive

4:40 - $1.54

$0.02 / GB


LP6005441-DNA_C06.srt.aln.bam - 206.8 GB

c4.large - 2/4 - A bit excessive but smaller may crash

14:27 - $3.43

$0.016 / GB



LP6005592-DNA_C05.srt.aln.bam - 126.9 GB

c4.large - 2/4 - A bit excessive but smaller may crash

8:55 - $2.22

$0.017 / GB
```



bamtofastq is single threaded. The prep work by the container is multithreaded so pre and post work are as fast as can be.






###	BWA MEM Bundle

Aligns to index in tar file, sorts, indexes, checks pipline output.



Testing on SGDP fastq pairs from previous step.





Usually runs out of memory

2023-01-27T01:41:52.832369661Z Error in piping. Pipe statuses: 137 0

Exit code of 137 is usually an out of memory error.



They suggest "Total memory parameter. For larger files, we suggest using 58 GB of memory and 32 CPU cores."



Align one sgdp to hg38 with bwa - 16thread/40gb (c5.4xlarge - 16/32) (c5.9xlarge - 36/72 $1.53)

C5.9xlarge

maxes out cpu usages

Load ave about 37

Mem (set to 70) about 18GB, jumped to 40GB, jumped again. crashed. I think I ran out of memory (set to 65GB as instance showed on 68.69 available, failed.) (set to 60, failed again)

Disk about 15% to 25%


M5.8xlarge (32/128 [124.47]) 32/110

Used 70GB memory

7.5 hours (spot failed so total ~10 hours, $13.59)


Generates a command line like ...

```
/bin/bash -c " export REF_CACHE=${PWD} &&  tar -tvf hg38.chrXYM_alts.tar 1>&2 && tar -xf hg38.chrXYM_alts.tar &&  bwa mem -R '@RG\tLB:LP6005441-DNA_D11\tPL:Illumina\tID:1\tSM:HGDP01078' -t 36 $(tar -tf hg38.chrXYM_alts.tar --wildcards '*.bwt' | rev | cut -c 5- | rev) /sbgenomics/workspaces/423a27ba-1083-4d9f-9093-da3b466edc32/tasks/07d7347b-e31f-4d98-aa9d-0ac5a2f6c08b/bwa-mem-bundle-0-7-17-cwl-1-2/LP6005441-DNA_D11.srt.aln_1.fq.gz /sbgenomics/workspaces/423a27ba-1083-4d9f-9093-da3b466edc32/tasks/07d7347b-e31f-4d98-aa9d-0ac5a2f6c08b/bwa-mem-bundle-0-7-17-cwl-1-2/LP6005441-DNA_D11.srt.aln_2.fq.gz  | bamsort index=1 threads=36 level=1 tmplevel=-1 inputformat=sam outputformat=bam SO=coordinate indexfilename=HGDP01078.bam.bai M=HGDP01078.sormadup_metrics.log > HGDP01078.bam ;declare -i pipe_statuses=(\${PIPESTATUS[*]});len=\${#pipe_statuses[@]};declare -i tot=0;echo \${pipe_statuses[*]};for (( i=0; i<\${len}; i++ ));do if [ \${pipe_statuses[\$i]} -ne 0 ];then tot=\${pipe_statuses[\$i]}; fi;done;if [ \$tot -ne 0 ]; then . Pipe statuses: \${pipe_statuses[*]};fi; if [ \$tot -ne 0 ]; then false;fi"
```

Outputs bam based on sample name NOT THE INPUT FILENAME!




```
m5.8xlarge

BAM / 32 / 110

LP6005441-DNA_D11.srt.aln_1.fq.gz - 31.0 GB
LP6005441-DNA_D11.srt.aln_2.fq.gz - 30.6 GB

Price: $13.59  Duration:  9 hours, 41 minutes

Maxed out at about 70GB memory.
```

$0.22 / GB FQ
$0.18 / GB BAM


```
m5.8xlarge

BAM / 32 / 110

LP6005441-DNA_C06.srt.aln_1.fq.gz - 82.9 GB
LP6005441-DNA_C06.srt.aln_2.fq.gz - 82.0 GB

Price: $33.62  Duration:  20 hours, 24 minutes

Maxed out at about 122 GB memory.
Not sure what the 110GB memory option that I passed is used for.
122GB is memory used by all, 110 is not part of the command line.
```

$0.20 / GB FQ
$0.16 / GB BAM


Both of these had their initial SPOT instances fail.

Good instances with this step depending on file size m5.8x




Not sure if the memory requirement is completely filesize related, or the number of threads has any impact.


Testing on r5.4xlarge. Same memory, but half the CPUs.
If maxes out at 90/95GB, file size biggest predictor.
If half that, thread count has major impact.

```
r5.4xlarge (16/128) ($1.008/hr)

BAM / 16 / 110

LP6005592-DNA_C05.srt.aln_1.fq.gz - 52.2 GB
LP6005592-DNA_C05.srt.aln_2.fq.gz - 52.1 GB
```





###	MELT SINGLE





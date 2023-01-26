

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













##	Python API Upload

```
python3 -m pip install --upgrade --user sevenbridges-python
```


```
#!/usr/bin/env python


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
#	0.01000673635349509		#  MAX?

upload.status
#'COMPLETED'

#	Surprisingly fast for 5GB
```

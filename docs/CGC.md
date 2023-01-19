

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















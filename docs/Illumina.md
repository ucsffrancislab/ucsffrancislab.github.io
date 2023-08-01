
#	Illumina


Data available on illumina.com


##	CLI


https://developer.basespace.illumina.com/docs/content/documentation/cli/cli-overview


Download basespace


```
wget https://launch.basespace.illumina.com/CLI/latest/amd64-linux/bs
```

Login to your basespace account on Illumina.com


```
bs auth
Please go to this URL to authenticate:  https://basespace.illumina.com/oauth/device?code=XXXXXXX
```

Open that URL in the browser ...


```
Welcome, George Wendt

bs list projects
```

```
bs list datasets
```




```
bs list projects
+-----------------------------------------+-----------+-------------+
|                  Name                   |    Id     |  TotalSize  |
+-----------------------------------------+-----------+-------------+
| MS_2x151nano_Project71_1-77_Gilly091219 | 141020880 | 546895506   |
| 072623_HH_55CATS-1                      | 394535155 | 78133865879 |
+-----------------------------------------+-----------+-------------+
```



```
bs download project -i 141020880

Completed files 4397 / 5314 [================================================================================================>--------------------]  82.74% 00m19s
```







```
bs download project -i 394535155
```










#	Box

Command line access to box from UCSF.

WebDAV is no longer supported.



##	NOTES

The max file size limitation is stated to be 50GB but we've found it to be much less,
at least when using curl to upload from the command line.

```
29949918869 - CS-4941-01A-01D-1465.bam - Succeeded
32556286377 - DU-5854-01A-11D-1703.bam - Succeeded
34011562949 - DB-5276-10A-01D-1465.bam - Succeeded
34553253061 - DH-5143-10A-01D-1465.bam - Failed
36992859467 - DU-5847-01A-11D-1703.bam - Failed 
```



##	Credentials

Accessing box via the command line will require getting special credentials from Box.

Log in to box using your UCSF credentials.

Click on the icon in the upper right, mine is a "GW", then click on "Account Settings".

Scroll down to the "Authentication" section. It should say something like "Because you are using Single Sign On (SSO), you will need to create a unique password to use external applications that do not support SSO." Follow the directions to create a password.


##	Password in `~/.netrc`

Add your username and new password to your `~/.netrc`.

This file may or may not exist already.

```
#machine dav.box.com login XXXXXXXXX@ucsf.edu password XXXXXXXX
machine ftp.box.com login XXXXXXXX@ucsf.edu password XXXXXXXX
```

You may also want to change its permissions ...
```
chmod 600 ~/.netrc
```



##	Example upload

Because the box "path" includes a space it MUST by parenthesesed or perhaps escaped.

With FTP instead of DAV, the dirs can be created during the upload command.

```
#BOX="https://dav.box.com/dav/Francis _Lab_Share/CLI_TEST"
BOX="ftps://ftp.box.com/Francis _Lab_Share/CLI_TEST"

#	Unnecessary
curl --ftp-create-dirs -netrc -X MKCOL "${BOX}/"

echo "Testing" > test_file.txt
curl --ftp-create-dirs -netrc -T test_file.txt "${BOX}/"
```

Note that the trailing "/" is essential.




##	Example download


###	curl

If not told otherwise, curl writes the received data to stdout. It can be instructed to instead save
that  data  into a local file, using the -o, --output or -O, --remote-name options. If curl is given
multiple URLs to transfer on the command line, it similarly needs multiple options for where to save
them.


```
#BOX="https://dav.box.com/dav/Francis _Lab_Share/CLI_TEST"
BOX="ftps://ftp.box.com/Francis _Lab_Share/CLI_TEST"

curl --ftp-create-dirs -netrc "${BOX}/test_file.txt" --output ./test_file.txt.download

cat test_file.txt.download 
Testing
```



Also see ...

https://ucsf-cbi.github.io/c4/transfers/ucsf-box.html

Their tutorial is a bit different.





###	wget

Apparently wget will look in .netrc automatically.


To download say a whole directory ...

Use wget / ftp

Add this slightly different line to your `.netrc`

```
machine ftp.box.com login XXXXXXXXXXXX@ucsf.edu password XXXXXXXX
```

Note that the path is a bit different here.

```
BOX="ftps://ftp.box.com/Francis _Lab_Share/Mayo-FrancisLab/Mayo/Observed"
wget --recursive "${BOX}"
```






###	rsync

I have yet to find a way to get rsync to use the .netrc credentials.








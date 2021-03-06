
#	Box

Command line access to box from UCSF.


##	Credentials

Accessing box via the command line will require getting special credentials from Box.

Log in to box using your UCSF credentials.

Click on the icon in the upper right, mine is a "GW", then click on "Account Settings".

Scroll down to the "Authentication" section. It should say something like "Because you are using Single Sign On (SSO), you will need to create a unique password to use external applications that do not support SSO." Follow the directions to create a password.


##	Password in `~/.netrc`

Add your username and new password to your `~/.netrc`.

This file may or may not exist already.

```
machine dav.box.com login George.Wendt@ucsf.edu password XXXXXXXX
```

You may also want to change its permissions ...
```
chmod 600 ~/.netrc
```



##	Example upload

Because the box "path" includes a space it MUST by parenthesed or perhaps escaped.

```
BOX="https://dav.box.com/dav/Francis _Lab_Share/CLI_TEST"

curl -netrc -X MKCOL "${BOX}/"

echo "Testing" > test_file.txt
curl -netrc -T test_file.txt "${BOX}/"
```

Note that the trailing "/" is essential.




##	Example download


If not told otherwise, curl writes the received data to stdout. It can be instructed to instead save
that  data  into a local file, using the -o, --output or -O, --remote-name options. If curl is given
multiple URLs to transfer on the command line, it similarly needs multiple options for where to save
them.


```
BOX="https://dav.box.com/dav/Francis _Lab_Share/CLI_TEST"

curl -netrc "${BOX}/test_file.txt" --output ./test_file.txt.download

cat test_file.txt.download 
Testing
```



Also see ...

https://ucsf-cbi.github.io/c4/transfers/ucsf-box.html

Their tutorial is a bit different.



To download say a whole directory ...

Use wget / ftp

Add this slightly different line to your `.netrc`

```
machine ftp.box.com login George.Wendt@ucsf.edu password XXXXXXXX
```

Note that the path is a bit different here.

```
BOX="ftps://ftp.box.com/Francis _Lab_Share/Mayo-FrancisLab/Mayo/Observed"
wget --recursive "${BOX}"
```





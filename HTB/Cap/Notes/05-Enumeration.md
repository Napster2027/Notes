# Nmap-
```bash
root@napster:~/Documents/HTB/Cap# nmap -sC -sV -oA nmap/initial 10.10.10.245
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-20 04:11 EDT
Nmap scan report for 10.10.10.245
Host is up (0.17s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    gunicorn
```


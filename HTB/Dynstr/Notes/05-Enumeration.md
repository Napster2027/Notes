# Nmap-

```bash
root@napster:~/Documents/HTB/Dynstr# nmap -sC -sV -oA nmap/initial 10.10.10.244
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-29 12:15 EDT
Nmap scan report for 10.10.10.244
Host is up (0.18s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Dyna DNS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```



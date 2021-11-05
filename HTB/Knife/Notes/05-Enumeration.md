# Nmap:
```bash
root@napster:~/Documents/HTB/Knife# nmap -sC -sV -oA nmap/initial 10.10.10.242
Starting Nmap 7.80 ( https://nmap.org ) at 2021-05-30 12:09 EDT
Nmap scan report for 10.10.10.242
Host is up (0.16s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kerne
```
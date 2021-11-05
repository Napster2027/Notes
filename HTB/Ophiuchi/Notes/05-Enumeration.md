# Nmap-
```bash
root@napster:~/Documents/HTB/Ophiuchi# nmap -sC -sV -oA nmap/initial  10.10.10.227
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-20 15:50 EDT
Nmap scan report for 10.10.10.227
Host is up (0.16s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http    Apache Tomcat 9.0.38
|_http-title: Parse YAML
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
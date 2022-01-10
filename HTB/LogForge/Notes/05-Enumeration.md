# # `Nmap` -

```bash
root@napster:~/Documents/HTB/LogForge# nmap -sC -sV -oA nmap/initial 10.10.11.138
Starting Nmap 7.80 ( https://nmap.org ) at 2022-01-07 10:39 EST
Nmap scan report for 10.10.11.138
Host is up (0.32s latency).
Not shown: 996 closed ports
PORT     STATE    SERVICE    VERSION
21/tcp   filtered ftp
22/tcp   open     ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http       Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Ultimate Hacking Championship
8080/tcp filtered http-proxy
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.90 seconds
```

Interestingly nmap shows Port `21` & `8080` filtered we will keep that in mind.
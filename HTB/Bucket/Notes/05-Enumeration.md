```bash
root@napster:~/Documents/HTB/Bucket# nmap -sC -sV 10.10.10.212
Starting Nmap 7.80 ( https://nmap.org ) at 2021-05-06 17:48 EDT
Nmap scan report for 10.10.10.212
Host is up (0.19s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://bucket.htb/
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.22 seconds
```

### >>Added http://bucket.htb/ to our host file-

-  10.10.10.212    bucket.htb


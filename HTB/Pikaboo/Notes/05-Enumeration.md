# # `Nmap` -

```bash
root@napster:~/Documents/HTB/Pikaboo# nmap -sC -sV -oA nmap/initial 10.10.10.249
Starting Nmap 7.80 ( https://nmap.org ) at 2021-07-21 11:33 EDT
Nmap scan report for 10.10.10.249
Host is up (0.17s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 17:e1:13:fe:66:6d:26:b6:90:68:d0:30:54:2e:e2:9f (RSA)
|   256 92:86:54:f7:cc:5a:1a:15:fe:c6:09:cc:e5:7c:0d:c3 (ECDSA)
|_  256 f4:cd:6f:3b:19:9c:cf:33:c6:6d:a5:13:6a:61:01:42 (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Pikaboo
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.39 seconds
```

- Port 21,22 & 80.
-  We have ftp on port 21 but no anonymous login nmap scan is usually very good and tell us if anonymous login is allowed or not. 

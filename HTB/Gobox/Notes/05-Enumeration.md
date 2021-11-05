# # `Nmap` -

```bash
root@napster:~/Documents/HTB/Gobox# nmap -sC -sV -oA nmap/initial 10.10.11.113
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-04 08:56 EDT
Nmap scan report for 10.10.11.113
Host is up (0.17s latency).
Not shown: 994 closed ports
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http       nginx
|_http-title: Hacking eSports | {{.Title}}
8080/tcp open     http       nginx
|_http-title: Hacking eSports | Home page
9000/tcp filtered cslistener
9001/tcp filtered tor-orport
9002/tcp filtered dynamid
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.07 seconds
```

The title looks odd.
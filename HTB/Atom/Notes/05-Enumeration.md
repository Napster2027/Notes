## >>> Nmap:

```bash
root@napster:~/Documents/HTB/Atom# nmap -sC -sV -oA nmap/initial 10.10.10.237                                                                                 
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-14 13:03 EDT                                                                                               
Nmap scan report for 10.10.10.237                                                                                                                             
Host is up (0.15s latency).                                                                                                                                   
Not shown: 996 filtered ports                                                                                                                                 
PORT    STATE SERVICE      VERSION                                                                                                                            
80/tcp  open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)                                                                            
| http-methods:                                                                                                                                               
|_  Potentially risky methods: TRACE                                                                                                                          
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27                                                                                         
|_http-title: Heed Solutions                                                   
135/tcp open  msrpc        Microsoft Windows RPC                                                                                                              
443/tcp open  ssl/http     Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-methods:                        
|_  Potentially risky methods: TRACE                                           
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27                                                                                         
|_http-title: Heed Solutions                                                   
| ssl-cert: Subject: commonName=localhost                                                                                                                     
| Not valid before: 2009-11-10T23:48:47                                        
|_Not valid after:  2019-11-08T23:48:47                                        
|_ssl-date: TLS randomness does not represent time                                                                                                            
| tls-alpn:                            
|_  http/1.1                           
445/tcp open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: ATOM; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## >>> Smb:

**On Enumerating Smb shares we found a pdf file-**

```bash
root@napster:~/Documents/HTB/Atom# smbclient \\\\10.10.10.237\\Software_Updates
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jun 14 13:23:25 2021
  ..                                  D        0  Mon Jun 14 13:23:25 2021
  client1                             D        0  Mon Jun 14 13:23:25 2021
  client2                             D        0  Mon Jun 14 13:23:25 2021
  client3                             D        0  Mon Jun 14 13:23:25 2021
  UAT_Testing_Procedures.pdf          A    35202  Fri Apr  9 07:18:08 2021

                4413951 blocks of size 4096. 1367280 blocks available
smb: \> ls -la
NT_STATUS_NO_SUCH_FILE listing \-la
smb: \> ls
  .                                   D        0  Mon Jun 14 13:23:25 2021
  ..                                  D        0  Mon Jun 14 13:23:25 2021
  client1                             D        0  Mon Jun 14 13:23:25 2021
  client2                             D        0  Mon Jun 14 13:23:25 2021
  client3                             D        0  Mon Jun 14 13:23:25 2021
  UAT_Testing_Procedures.pdf          A    35202  Fri Apr  9 07:18:08 2021

                4413951 blocks of size 4096. 1367280 blocks available
smb: \> get UAT_Testing_Procedures.pdf 
getting file \UAT_Testing_Procedures.pdf of size 35202 as UAT_Testing_Procedures.pdf (45.2 KiloBytes/sec) (average 45.2 KiloBytes/sec)
smb: \> exit
```

## >>> Pdf File:

![[Pasted image 20210614140951.png]]

![[Pasted image 20210614141136.png]]



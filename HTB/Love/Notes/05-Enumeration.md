# Nmap-

```bash
root@napster:~/Documents/HTB/Love# nmap -sC -sV -oA nmap/initial 10.10.10.239                                                                                 
Starting Nmap 7.80 ( https://nmap.org ) at 2021-07-01 18:09 EDT                                                                                               
Nmap scan report for 10.10.10.239                                                                                                                             
Host is up (0.16s latency).                                                                                                                                   
Not shown: 993 closed ports                                                                                                                                   
PORT     STATE SERVICE      VERSION                                                                                                                           
80/tcp   open  http         Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)                                                                           
| http-cookie-flags:                                                                                                                                          
|   /:                                                                                                                                                        
|     PHPSESSID:                                                                                                                                              
|_      httponly flag not set                                                                                                                                 
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27                                                                                         
|_http-title: Voting System using PHP                                                                                                                         
135/tcp  open  msrpc        Microsoft Windows RPC                                                                                                             
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn                                                                                                     
443/tcp  open  ssl/http     Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)                                                                                   
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27                                                                                         
|_http-title: 403 Forbidden                                                                                                                                   
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in                                          
| Not valid before: 2021-01-18T14:00:16                                                                                                                       
|_Not valid after:  2022-01-18T14:00:16                                                                                                                       
|_ssl-date: TLS randomness does not represent time                                                                                                            
| tls-alpn:                                                                                                                                                   
|_  http/1.1                                                                                                                                                  
445/tcp  open  microsoft-ds Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)                                                                          
3306/tcp open  mysql?
| fingerprint-strings: 
|   LANDesk-RC, RPCCheck: 
|_    Host '10.10.15.4' is not allowed to connect to this MariaDB server
5000/tcp open  http         Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
```

Output-
1. Windows 10 Pro 19042
2. commonName=staging.love.htb(on ssl)
3. Mysql 3306
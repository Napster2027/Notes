# Nmap:
```bash
root@napster:~/Documents/HTB/Explore# nmap -sC -sV -oA nmap/initial 10.10.10.247
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-28 10:07 EDT
Nmap scan report for 10.10.10.247
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT     STATE    SERVICE VERSION
2222/tcp open     ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-SSH Server - Banana Studio
| ssh-hostkey: 
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
5555/tcp filtered freeciv
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port2222-TCP:V=7.80%I=7%D=6/28%Time=60D9D792%P=x86_64-pc-linux-gnu%r(NU
SF:LL,24,"SSH-2\.0-SSH\x20Server\x20-\x20Banana\x20Studio\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.49 seconds
```

Port 5555 (Android Debug Bridge Daemon (ADBD) 		[Android Device]

### Full Scan:
```bash
root@napster:~/Documents/HTB/Explore# nmap -p- -oA nmap/full 10.10.10.247                                                                                     
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-28 10:22 EDT                                                                                               
Nmap scan report for 10.10.10.247                                                                                                                             
Host is up (0.19s latency).                                                                                                                                   
Not shown: 65530 closed ports                                                                                                                                 
PORT      STATE    SERVICE                                                                                                                                    
2222/tcp  open     EtherNetIP-1                                                                                                                               
5555/tcp  filtered freeciv                                                                                                                                    
38511/tcp open     unknown                                                                                                                                    
42135/tcp open     unknown                                                                                                                                    
59777/tcp open     unknown 
```

```bash

root@napster:~/Documents/HTB/Explore# nmap -sC -sV -p 2222,5555,38511,42135,59777  10.10.10.247                                                       [78/222]
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-28 10:43 EDT                                                                                               
Nmap scan report for 10.10.10.247                                                                                                                             
Host is up (0.19s latency).                                                                                                                                   
                                                                                                                                                              
PORT      STATE    SERVICE VERSION                                                                                                                            
2222/tcp  open     ssh     (protocol 2.0)                                                                                                                     
| fingerprint-strings:                                                                                                                                        
|   NULL:                                                                                                                                                     
|_    SSH-2.0-SSH Server - Banana Studio                                                                                                                      
| ssh-hostkey:                                                                                                                                                
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)                                                                                                
5555/tcp  filtered freeciv                                                                                                                                    
38511/tcp open     unknown
<SNIPPET>
42135/tcp open     http    ES File Explorer Name Response httpd
|_http-title: Site doesn't have a title (text/html).
59777/tcp open     http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
|_http-title: Site doesn't have a title (text/plain).
```


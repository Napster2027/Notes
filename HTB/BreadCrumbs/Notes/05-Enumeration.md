# Nmap -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# nmap -sC -sV -oA nmap/initial 10.10.10.228
# Nmap 7.80 scan initiated Wed Jul 28 12:31:38 2021 as: nmap -sC -sV -oA nmap/initial 10.10.10.228
Nmap scan report for 10.10.10.228
Host is up (0.18s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 9d:d0:b8:81:55:54:ea:0f:89:b1:10:32:33:6a:a7:8f (RSA)
|   256 1f:2e:67:37:1a:b8:91:1d:5c:31:59:c7:c6:df:14:1d (ECDSA)
|_  256 30:9e:5d:12:e3:c6:b7:c6:3b:7e:1e:e7:89:7e:83:e4 (ED25519)
80/tcp   open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, HTTPOptions, NCP, NotesRPC, RTSPRequest, TLSSessionReq, TerminalServerCookie, WMSRequest: 
|_    Host '10.10.14.196' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.80%I=7%D=7/28%Time=6101867C%P=x86_64-pc-linux-gnu%r(HT
SF:TPOptions,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x20is\x20not\
SF:x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(RTSP
SF:Request,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x20is\x20not\x2
SF:0allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(DNSSta
SF:tusRequestTCP,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x20is\x20
SF:not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(
SF:TerminalServerCookie,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x2
SF:0is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20serv
SF:er")%r(TLSSessionReq,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x2
SF:0is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20serv
SF:er")%r(NCP,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x20is\x20not
SF:\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(Not
SF:esRPC,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x20is\x20not\x20a
SF:llowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(WMSReque
SF:st,4B,"G\0\0\x01\xffj\x04Host\x20'10\.10\.14\.196'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1h00m00s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-28T15:32:08
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jul 28 12:32:16 2021 -- 1 IP address (1 host up) scanned in 38.75 seconds
```

- Windows Machine
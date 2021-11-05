# Nmap -

```bash
# Nmap 7.80 scan initiated Sat Jul 10 15:18:51 2021 as: nmap -sC -sV -oA nmap/initial 10.10.10.250
Nmap scan report for 10.10.10.250
Host is up (0.16s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
443/tcp  open  ssl/http   nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Seal Market
| ssl-cert: Subject: commonName=seal.htb/organizationName=Seal Pvt Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-05-05T10:24:03
|_Not valid after:  2022-05-05T10:24:03
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
8080/tcp open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 401 Unauthorized
|     Date: Sat, 10 Jul 2021 19:19:03 GMT
|     Set-Cookie: JSESSIONID=node013qfjjlsxitsru3p2wkitjxeh90.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 0
|   GetRequest: 
|     HTTP/1.1 401 Unauthorized
|     Date: Sat, 10 Jul 2021 19:19:02 GMT
|     Set-Cookie: JSESSIONID=node0x77yu67wy1ga1wcw60aih55w88.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Sat, 10 Jul 2021 19:19:02 GMT
|     Set-Cookie: JSESSIONID=node015brjtpebx8pz1teov8nl821bu89.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Allow: GET,HEAD,POST,OPTIONS
|     Content-Length: 0
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 505 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 505</h1><pre>reason: Unknown Version</pre>
|   Socks4: 
|     HTTP/1.1 400 Illegal character CNTL=0x4
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x4</pre>
|   Socks5: 
|     HTTP/1.1 400 Illegal character CNTL=0x5
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x5</pre>
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.80%I=7%D=7/10%Time=60E9F2A5%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,F4,"HTTP/1\.1\x20401\x20Unauthorized\r\nDate:\x20Sat,\x2010\x2
SF:0Jul\x202021\x2019:19:02\x20GMT\r\nSet-Cookie:\x20JSESSIONID=node0x77yu
SF:67wy1ga1wcw60aih55w88\.node0;\x20Path=/;\x20HttpOnly\r\nExpires:\x20Thu
SF:,\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nContent-Type:\x20text/html
SF:;charset=utf-8\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,10A,"HTT
SF:P/1\.1\x20200\x20OK\r\nDate:\x20Sat,\x2010\x20Jul\x202021\x2019:19:02\x
SF:20GMT\r\nSet-Cookie:\x20JSESSIONID=node015brjtpebx8pz1teov8nl821bu89\.n
SF:ode0;\x20Path=/;\x20HttpOnly\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x
SF:2000:00:00\x20GMT\r\nContent-Type:\x20text/html;charset=utf-8\r\nAllow:
SF:\x20GET,HEAD,POST,OPTIONS\r\nContent-Length:\x200\r\n\r\n")%r(RTSPReque
SF:st,AD,"HTTP/1\.1\x20505\x20Unknown\x20Version\r\nContent-Type:\x20text/
SF:html;charset=iso-8859-1\r\nContent-Length:\x2058\r\nConnection:\x20clos
SF:e\r\n\r\n<h1>Bad\x20Message\x20505</h1><pre>reason:\x20Unknown\x20Versi
SF:on</pre>")%r(FourOhFourRequest,F5,"HTTP/1\.1\x20401\x20Unauthorized\r\n
SF:Date:\x20Sat,\x2010\x20Jul\x202021\x2019:19:03\x20GMT\r\nSet-Cookie:\x2
SF:0JSESSIONID=node013qfjjlsxitsru3p2wkitjxeh90\.node0;\x20Path=/;\x20Http
SF:Only\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nCon
SF:tent-Type:\x20text/html;charset=utf-8\r\nContent-Length:\x200\r\n\r\n")
SF:%r(Socks5,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x5\r\nC
SF:ontent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r
SF:\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason
SF::\x20Illegal\x20character\x20CNTL=0x5</pre>")%r(Socks4,C3,"HTTP/1\.1\x2
SF:0400\x20Illegal\x20character\x20CNTL=0x4\r\nContent-Type:\x20text/html;
SF:charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n
SF:\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\
SF:x20CNTL=0x4</pre>")%r(RPCCheck,C7,"HTTP/1\.1\x20400\x20Illegal\x20chara
SF:cter\x20OTEXT=0x80\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\n
SF:Content-Length:\x2071\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message
SF:\x20400</h1><pre>reason:\x20Illegal\x20character\x20OTEXT=0x80</pre>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jul 10 15:19:24 2021 -- 1 IP address (1 host up) scanned in 32.63 seconds
```

- Port 443(HTTPS) and 8080 opened.
- Found hostname `seal.htb`

Edit your hostfile and add the host -

```bash
10.10.10.250    seal.htb
```
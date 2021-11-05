# Nmap-
1. DNS:dms-pit.htb

```bash
root@napster:~/Documents/HTB/Pit# nmap -sC -sV -oA nmap/initial 10.10.10.241
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-19 02:27 EDT                                                                                               
Nmap scan report for 10.10.10.241                                                                                                                             
Host is up (0.19s latency).                                                                                                                                   
Not shown: 997 filtered ports                                                                                                                                 
PORT     STATE SERVICE         VERSION                                                                                                                        
22/tcp   open  ssh             OpenSSH 8.0 (protocol 2.0)                                                                                                     
| ssh-hostkey:                                                                                                                                                
|   3072 6f:c3:40:8f:69:50:69:5a:57:d7:9c:4e:7b:1b:94:96 (RSA)                                                                                                
|   256 c2:6f:f8:ab:a1:20:83:d1:60:ab:cf:63:2d:c8:65:b7 (ECDSA)                                                                                               
|_  256 6b:65:6c:a6:92:e5:cc:76:17:5a:2f:9a:e7:50:c3:50 (ED25519)                                                                                             
80/tcp   open  http            nginx 1.14.1                                                                                                                   
|_http-server-header: nginx/1.14.1                                                                                                                            
|_http-title: Test Page for the Nginx HTTP Server on Red Hat Enterprise Linux                                                                                 
9090/tcp open  ssl/zeus-admin?
ssl-cert: Subject: commonName=dms-pit.htb/organizationName=4cd9329523184b0ea52ba0d20a1a6f92/countryName=US
| Subject Alternative Name: DNS:dms-pit.htb, DNS:localhost, IP Address:127.0.0.1
```

We didn't found anything on the port 80 just a default nginx page.Port 9090 was a login page(cockpit) tried some cve's but didnt worked:(

For further enumeration tried to look for udp ports found snmp-

```bash
root@napster:~/Documents/HTB/Pit# nping --udp -c 2 -p 161 10.10.10.241

Starting Nping 0.7.80 ( https://nmap.org/nping ) at 2021-06-19 03:24 EDT
SENT (0.0408s) UDP 10.10.14.16:53 > 10.10.10.241:161 ttl=64 id=8531 iplen=28 
SENT (1.0423s) UDP 10.10.14.16:53 > 10.10.10.241:161 ttl=64 id=8531 iplen=28 
 
Max rtt: N/A | Min rtt: N/A | Avg rtt: N/A
Raw packets sent: 2 (56B) | Rcvd: 0 (0B) | Lost: 2 (100.00%)
Nping done: 1 IP address pinged in 2.08 seconds
```

### Nmap(SNMP)-

```bash
root@napster:~/Documents/HTB/Pit# nmap -sU -p 161,162 -sV 10.10.10.241
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-19 03:26 EDT
Nmap scan report for dms-pit.htb (10.10.10.241)
Host is up (0.17s latency).

PORT    STATE    SERVICE  VERSION
161/udp open     snmp     SNMPv1 server; net-snmp SNMPv3 server (public)
162/udp filtered snmptrap
Service Info: Host: pit.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.79 seconds
```

We got the version information of SNMP and it also disclosed it is using `Public` community string for authentication.
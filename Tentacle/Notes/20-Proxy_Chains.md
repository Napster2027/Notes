# Enumeration-

Using proxychains to scan the new subdomains we found-

```bash
strict_chain
proxy_dns
[ProxyList]
http 10.10.10.224 3128
http 127.0.0.1 3128
```

First going through 10.10.10.224:3128 and then through 127.0.0.1:3128

# Scanning through Proxychains(Double Proxy)-
### `10.197.243.77-->ns/proxy.realcorp.htb`

```bash
root@napster:~/Documents/HTB/Tentacle# proxychains nmap -sT -Pn -n 10.197.243.77 -oA nmap/10.197.243.77
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-26 12:15 EDT
Nmap scan report for 10.197.243.77
Host is up (0.59s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
88/tcp   open  kerberos-sec
464/tcp  open  kpasswd5
749/tcp  open  kerberos-adm
3128/tcp open  squid-http
```

-sT: Full Tcp Scan as we are scanning through proxychains.
-Pn: Port Scan Only
-n: Disable DNS

This also has Squid Proxy & it's name is also ns/proxy.realcorp.htb.So maybe using this proxy we can access something 

### `10.197.243.31-->wpad.realcorp.htb`
```bash
root@napster:~/Documents/HTB/Tentacle# proxychains nmap -sT -Pn -n 10.197.243.31
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-26 13:05 EDT
Nmap scan report for 10.197.243.31
Host is up (0.90s latency).
All 1000 scanned ports on 10.197.243.31 are closed
```


# Triple Proxy-
```bash
strict_chain
proxy_dns
[ProxyList]
http 10.10.10.224 3128
http 127.0.0.1 3128
http 10.197.243.77 3128
```

### `10.197.243.31`

```bash
root@napster:~/Documents/HTB/Tentacle# proxychains nmap -sT -Pn -n 10.197.243.31 -oA nmap/10.197.243.31
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-26 13:23 EDT
Nmap scan report for 10.197.243.31
Host is up (0.64s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
464/tcp  open  kpasswd5
749/tcp  open  kerberos-adm
3128/tcp open  squid-http
```

Squid Proxy(3128) again running but we also have http running on port(80).

The name of the domain is wpad.realcorp.htb.WPAD or `Web Proxy Auto-Discovery Protocol` is a way for clients to automatically find and use a proxy configuration file. The client gets the WPAD URL either using DHCP or DNS.

Adding `10.197.243.31   wpad.realcorp.htb` to `/etc/hosts`

There is a `wpad.dat` file in the web root:

```bash
root@napster:~/Documents/HTB/Tentacle# proxychains curl http://wpad.realcorp.htb/wpad.dat
ProxyChains-3.1 (http://proxychains.sf.net)
function FindProxyForURL(url, host) {
    if (dnsDomainIs(host, "realcorp.htb"))
        return "DIRECT";
    if (isInNet(dnsResolve(host), "10.197.243.0", "255.255.255.0"))
        return "DIRECT"; 
    if (isInNet(dnsResolve(host), "10.241.251.0", "255.255.255.0"))
        return "DIRECT"; 
 
    return "PROXY proxy.realcorp.htb:3128";
}
```

There is a new IP address range `10.241.251.0`


### `10.241.251.0`

Using nmap to find all the hosts-

```bash
root@napster:~/Documents/HTB/Tentacle# nmap -sL --dns-servers 10.10.10.224 10.241.251.0/24 | grep -F '('
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-26 16:57 EDT
Nmap scan report for srvpod01.realcorp.htb (10.241.251.113)
Nmap done: 256 IP addresses (0 hosts up) scanned in 0.80 seconds
```

Found new domain `10.241.251.113-->srvpod01.realcorp.htb`

# Nmap (srvpod01.realcorp.htb)
### `10.241.251.113`

```bash
root@napster:~/Documents/HTB/Tentacle# proxychains nmap -sT -Pn -n 10.241.251.113 -oA nmap/10.241.251.113
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.80 ( https://nmap.org ) at 2021-06-26 17:18 EDT
Nmap scan report for 10.241.251.113
Host is up (0.66s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
25/tcp open  smtp
```
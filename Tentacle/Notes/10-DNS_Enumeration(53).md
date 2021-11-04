# Enumeration-

```bash
root@napster:~/Documents/HTB/Tentacle# dig axfr @10.10.10.224

; <<>> DiG 9.11.5-P4-3-Debian <<>> axfr @10.10.10.224
; (1 server found)
;; global options: +cmd
;; Query time: 292 msec
;; SERVER: 10.10.10.224#53(10.10.10.224)
;; WHEN: Sat Jun 26 09:40:36 EDT 2021
;; MSG SIZE  rcvd: 56

root@napster:~/Documents/HTB/Tentacle# dig axfr @10.10.10.224 realcorp.htb

; <<>> DiG 9.11.5-P4-3-Debian <<>> axfr @10.10.10.224 realcorp.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

# Dnsenum-

Using dnsenum to bruteforce subdomains over DNS-

```bash
root@napster:~/Documents/HTB/Tentacle# dnsenum --dnsserver 10.10.10.224 -f /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -o subdomains.txt realcorp.htb
-----   realcorp.htb   -----


Host's addresses:
__________________



Name Servers:
______________

ns.realcorp.htb.                         259200   IN    A        10.197.243.77


Brute forcing with /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:
__________________________________________________________________________________________________

ns.realcorp.htb.                         259200   IN    A        10.197.243.77
proxy.realcorp.htb.                      259200   IN    CNAME    ns.realcorp.htb.
ns.realcorp.htb.                         259200   IN    A        10.197.243.77
wpad.realcorp.htb.                       259200   IN    A        10.197.243.31
```


### Found 3 subdomains-
### 1. ns/proxy.realcorp.htb-->10.197.243.77
### 2. wpad.realcorp.htb-->10.197.243.31



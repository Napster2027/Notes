# Enumeration-

\>>> Looked for any  `zone transfers`-

```bash
root@napster:~/Documents/HTB/Dynstr# dig axfr @10.10.10.244

; <<>> DiG 9.11.5-P4-3-Debian <<>> axfr @10.10.10.244
; (1 server found)
;; global options: +cmd
;; Query time: 171 msec
;; SERVER: 10.10.10.244#53(10.10.10.244)
;; WHEN: Tue Jun 29 12:26:14 EDT 2021
;; MSG SIZE  rcvd: 56

root@napster:~/Documents/HTB/Dynstr# dig axfr @10.10.10.244 dyna.htb

; <<>> DiG 9.11.5-P4-3-Debian <<>> axfr @10.10.10.244 dyna.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

\>>> Looked for additional DNS Info available-

```bash
root@napster:~/Documents/HTB/Dynstr# dig ANY @10.10.10.244 dyna.htb

; <<>> DiG 9.11.5-P4-3-Debian <<>> ANY @10.10.10.244 dyna.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25512
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: eda362906c8def6e0100000060db4eadf58bb9b3b019693a (good)
;; QUESTION SECTION:
;dyna.htb.                      IN      ANY

;; ANSWER SECTION:
dyna.htb.               60      IN      SOA     dns1.dyna.htb. hostmaster.dyna.htb. 2021030304 21600 3600 604800 60
dyna.htb.               60      IN      NS      dns1.dyna.htb.

;; ADDITIONAL SECTION:
dns1.dyna.htb.          60      IN      A       127.0.0.1

;; Query time: 169 msec
;; SERVER: 10.10.10.244#53(10.10.10.244)
;; WHEN: Tue Jun 29 12:47:40 EDT 2021
;; MSG SIZE  rcvd: 147
```

Found  two Subdomains-

`dns1.dyna.htb hostmaster.dyna.htb`

Adding them to host file-				[/etc/hosts]

`10.10.10.244    dns1.dyna.htb hostmaster.dyna.htb`

# Enumeration-
```bash
root@napster:~/Documents/HTB/Pit# perl snmpbw.pl 10.10.10.241 public 2 1
SNMP query:       10.10.10.241
Queue count:      0
SNMP SUCCESS:     10.10.10.241
```
[Output File:10.10.10.241.snmp]-
```bash
.1.3.6.1.2.1.1.1.0 = STRING: "Linux pit.htb 4.18.0-240.22.1.el8_3.x86_64 #1 SMP Thu Apr 8 19:01:30 UTC 2021 x86_64"
.1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.8072.3.2.10
.1.3.6.1.2.1.1.3.0 = Timeticks: (716862) 1:59:28.62
.1.3.6.1.2.1.1.4.0 = STRING: "Root <root@localhost> (configure /etc/snmp/snmp.local.conf)"
.1.3.6.1.2.1.1.5.0 = STRING: "pit.htb"
.1.3.6.1.2.1.1.6.0 = STRING: "Unknown (edit /etc/snmp/snmpd.conf)"
<SNIP>
.1.3.6.1.4.1.2021.2.1.1.1 = INTEGER: 1
.1.3.6.1.4.1.2021.2.1.2.1 = STRING: "nginx"
.1.3.6.1.4.1.2021.2.1.3.1 = INTEGER: 1
.1.3.6.1.4.1.2021.2.1.4.1 = INTEGER: 0
.1.3.6.1.4.1.2021.2.1.5.1 = INTEGER: 3
.1.3.6.1.4.1.2021.2.1.100.1 = INTEGER: 0
.1.3.6.1.4.1.2021.2.1.102.1 = INTEGER: 0
.1.3.6.1.4.1.2021.2.1.103.1 = ""
.1.3.6.1.4.1.2021.9.1.1.1 = INTEGER: 1
.1.3.6.1.4.1.2021.9.1.1.2 = INTEGER: 2
.1.3.6.1.4.1.2021.9.1.2.1 = STRING: "/"
.1.3.6.1.4.1.2021.9.1.2.2 = STRING: "/var/www/html/seeddms51x/seeddms"
.1.3.6.1.4.1.2021.9.1.3.1 = STRING: "/dev/mapper/cl-root"
.1.3.6.1.4.1.2021.9.1.3.2 = STRING: "/dev/mapper/cl-seeddms"
<SNIP>
.1.3.6.1.4.1.8072.1.3.2.2.1.2.10.109.111.110.105.116.111.114.105.110.103 = STRING: "/usr/bin/monitor"
.1.3.6.1.4.1.8072.1.3.2.2.1.3.10.109.111.110.105.116.111.114.105.110.103 = ""
.1.3.6.1.4.1.8072.1.3.2.2.1.4.10.109.111.110.105.116.111.114.105.110.103 = ""
<SNIP>
.1.3.6.1.4.1.8072.1.3.2.4.1.2.10.109.111.110.105.116.111.114.105.110.103.25 = STRING: "Login Name           SELinux User         MLS/MCS Range        Service"
.1.3.6.1.4.1.8072.1.3.2.4.1.2.10.109.111.110.105.116.111.114.105.110.103.26 = ""
.1.3.6.1.4.1.8072.1.3.2.4.1.2.10.109.111.110.105.116.111.114.105.110.103.27 = STRING: "__default__          unconfined_u         s0-s0:c0.c1023       *"
.1.3.6.1.4.1.8072.1.3.2.4.1.2.10.109.111.110.105.116.111.114.105.110.103.28 = STRING: "michelle             user_u               s0                   *"
.1.3.6.1.4.1.8072.1.3.2.4.1.2.10.109.111.110.105.116.111.114.105.110.103.29 = STRING: "root                 unconfined_u         s0-s0:c0.c1023       *"
```

### Interesting things-
1. "/var/www/html/seeddms51x/seeddms"		[Location on web]
2. "/usr/bin/monitor"											[Location on disk]
3.  "michelle"														 [User]


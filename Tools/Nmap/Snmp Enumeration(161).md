# # `Commands` -

1. <mark style="background: #3D7EFFA6;">Scanning for SNMP :</mark>

```bash
nmap -sU --open -p 161 10.11.1.100
```

2. <mark style="background: #3D7EFFA6;">Automated Enumeration :</mark>

```bash
perl snmpbw.pl 10.10.11.107 public 2 1
```

```bash
snmpwalk -v1 -c public 10.10.10.241
```

```bash
snmpwalk -v2c -c public 10.10.11.193
```

 `snmpbulkwalk` (which is also installed with `apt install snmp`). Instead of making SNMP requests for each item OID (item) as `snmpwalk` does, `snmpbulkwalk` makes bulk requests, so it gets the same data 10 times faster:

```bash
snmpbulkwalk -v2c -c internal 10.10.11.193
```


3. <mark style="background: #3D7EFFA6;">Some interesting MIB values :</mark>

|1.3.6.1.2.1.25.1.6.0 |System Processes|
|--------------------|--------------------|
|1.3.6.1.2.1.25.4.2.1.2 |Running Programs|
|1.3.6.1.2.1.25.4.2.1.4| Processes Path |
|1.3.6.1.2.1.25.2.3.1.4 |Storage Units |
|1.3.6.1.2.1.25.6.3.1.2 |Software Name |
|1.3.6.1.4.1.77.1.2.25 |User Accounts |
|1.3.6.1.2.1.6.13.1.3 |TCP Local Ports |

For example Enumerating Windows Users -

```bash
snmpwalk -c public -v1 10.10.10.100 1.3.6.1.4.1.77.1.2.25
```

4. <mark style="background: #3D7EFFA6;">Bruteforce Community Strings :</mark>

In SNMP, community strings are kind of like a combination of username and password. The default one is “public”, but there can be others with different levels of access.

Tools like [onesixtyone](https://github.com/trailofbits/onesixtyone) and [SNMP-Brute](https://github.com/SECFORCE/SNMP-Brute) are made to brute force community strings :

```bash
python /opt/SNMP-Brute/snmpbrute.py -t 10.10.11.193
```

5. <mark style="background: #3D7EFFA6;">Configuration :</mark>

Config file located at -

```bash
/etc/snmp/snmpd.conf
```

There is bunch of commented lines, which you can remove with `grep -v "^#"`, and a bunch of empty lines, you can remove with `grep .`

```bash
cat snmpd.conf | grep -v "^#" | grep .
```


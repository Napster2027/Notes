## # `Basic Scan` -

```bash
nmap -sC -sV -A -oA nmap/initial 10.10.10.100					[make a folder called nmap]
```

- sC : Default Scripts
- sV : Enumerate versions 
- oA : Output All formats to the file

## # `All Ports Scan` -

```bash
nmap -p- 10.10.10.100
```

## # `Udp Scan` -

Normally, nmap doesnt scan for UDP ports you have to specify it or do it separately.

```bash
 nmap -sU 10.10.0.100
```

It takes a lot of time to scan UDP ports one trick is to do it the following way -

```bash
nmap -sU --min-rate 10000 10.10.0.100
```

## # `Eternal Blue Check` -

```bash
nmap -A -p445 –script vuln 10.10.10.40
nmap -p445 --script smb-vuln-ms17-010 10.10.10.40
```

All nmap scripts are located under -

```bash
/usr/share/nmap/scripts/
```

## # `Host Identification over DNS` -

```bash
nmap -sL --dns-servers 10.10.10.224 10.197.243.0/24 | grep -F '('
```

- [Reference: Tentacles(HTB)]

## # `Finding host in a Subnet One Liner` -

Using Ping -

```bash
for i in {1..255}; do (ping -c 1 172.17.0.${i} | grep "bytes from" | grep -v "Unreachable" &); done;
```

```bash
for i in {1..255};do(ping -c 1 172.16.1.$i|grep "bytes from"|cut -d' ' -f4|tr -d ':' &);done
```

```bash
(for /L %a IN (1,1,254) DO ping /n 1 /w 1 172.16.2.%a) | find "Reply"
```

Using Dig -

```bash
for i in {1..255}; do res=$(dig +short -x 172.69.0.${i}); if [ ! -z "$res" ]; then echo "172.69.0.${i}  $res"; fi; done
```

## # `Finding ports One Liner` -

```bash
export ip=172.16.1.13; for port in $(seq 1 65535); do timeout 0.01 bash -c "</dev/tcp/$ip/$port && echo Port open $port || echo Port closed $port > /dev/null" 2>/dev/null; done
```

## # `Filtered Vs Closed Ports` -

You can use ==--reason== in nmap -

```bash
nmap -sU --min-rate 10000 --reason 10.10.0.100
```

- ***==Filtered means admin-prohibited==*** : firewall blocking it.

- ***==Closed means port unreachable==*** : OS telling port is closed.
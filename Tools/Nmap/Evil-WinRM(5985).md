`Evil-winrm` is the best tool for connecting to WinRM from a Linux host.

# # `Commands` -

1.  Simple Login via Password -

```bash
root@napster:/opt/winrm-brute# evil-winrm -u Chase -p 'Q4)sJu\Y8qz*A3?d' -i 10.10.10.149
```

2. Login via certificate -

```bash
evil-winrm -i timelapse.htb -S -k legacyy_dev_auth.key -c legacyy_dev_auth.crt
```

-   `-S` - Enable SSL, because I’m connecting to 5986;
-   `-c legacyy_dev_auth.crt` - provide the public key certificate
-   `-k legacyy_dev_auth.key` - provide the private key
-   `-i timelapse.htb` - host to connect to


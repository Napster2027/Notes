# # `Enumeration` -

![[Pasted image 20210904090222.png]]


Title on Browser -

![[Pasted image 20210904090406.png]]

It looks like some SSTI challenge from the the title page.

# # `/index.html` -

Visiting /index.html we see a test text -

![[Pasted image 20210904090642.png]]

# # `Directory Listing` -

```bash
root@napster:~/Documents/HTB/Gobox# feroxbuster -u http://10.10.11.113/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -x php -o directory.txt                                  

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher π€                 ver: 2.3.3
ββββββββββββββββββββββββββββ¬ββββββββββββββββββββββ
 π―  Target Url            β http://10.10.11.113/
 π  Threads               β 50
 π  Wordlist              β /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
 π  Status Codes          β [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 π₯  Timeout (secs)        β 7
 π¦‘  User-Agent            β feroxbuster/2.3.3
 π  Config File           β /etc/feroxbuster/ferox-config.toml
 πΎ  Output File           β directory.txt
 π²  Extensions            β [php]
 π  Recursion Depth       β 4
ββββββββββββββββββββββββββββ΄ββββββββββββββββββββββ
 π  Press [ENTER] to use the Scan Cancel Menuβ’
ββββββββββββββββββββββββββββββββββββββββββββββββββ
301        7l       11w      162c http://10.10.11.113/css
200       54l      190w        0c http://10.10.11.113/index.php
200       54l      190w        0c http://10.10.11.113/
403        7l        9w      146c http://10.10.11.113/css/
```

Nothing Interesting.


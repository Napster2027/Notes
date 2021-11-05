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
by Ben "epi" Risher 🤓                 ver: 2.3.3
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.11.113/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.3.3
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💾  Output File           │ directory.txt
 💲  Extensions            │ [php]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Cancel Menu™
──────────────────────────────────────────────────
301        7l       11w      162c http://10.10.11.113/css
200       54l      190w        0c http://10.10.11.113/index.php
200       54l      190w        0c http://10.10.11.113/
403        7l        9w      146c http://10.10.11.113/css/
```

Nothing Interesting.


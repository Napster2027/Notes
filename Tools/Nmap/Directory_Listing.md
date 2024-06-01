# # `Commands` -

1. Go-to-Command for directory enumeration using <mark style="background: #3D7EFFA6;">Gobuster</mark> -

```bash
gobuster dir -u http://10.10.10.6/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt -t 50
```

```bash
gobuster dir -u http://10.10.11.138/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster.txt -t 20
```

---

2. Directory Brute force using <mark style="background: #3D7EFFA6;">Feroxbuster</mark> -

- Go to command prototype -

```bash
feroxbuster -u http://mailroom.htb -x php
```

- For API's -

```bash
feroxbuster -u http://10.10.11.162 --force-recursion -C 404,405 -m GET,POST
```

`--force-recursion` will recurse down endpoints even if they don’t act like directories, which can be useful on APIs. `-m GET,POST` will test both kinds of HTTP requests, as for APIs one may exist where the other doesn’t. Filter out only 404s and 405s, based on a quick run with no filters and seeing those are loud.
## # `Directory Brute` -

- <mark style="background: #3D7EFFA6;">Go to command prototype</mark> -

```bash
feroxbuster -u http://mailroom.htb -x php
```

```bash
feroxbuster -u http://10.10.11.212 -x php,html -C 400,502 --no-recursion --dont-extract-links
```

running with `--no-recursion` and `--dont-extract-links` , as both of these generate a ton of errors that aren’t useful.

- <mark style="background: #3D7EFFA6;">For API's</mark> -

```bash
feroxbuster -u http://10.10.11.162 --force-recursion -C 404,405 -m GET,POST
```

`--force-recursion` will recurse down endpoints even if they don’t act like directories, which can be useful on APIs. `-m GET,POST` will test both kinds of HTTP requests, as for APIs one may exist where the other doesn’t. Filter out only 404s and 405s, based on a quick run with no filters and seeing those are loud.

- <mark style="background: #3D7EFFA6;">Fuzz Request Methods</mark> -

```bash
feroxbuster -u http://proxy.gofer.htb -m GET,POST,PUT,OPTIONS,CONNECT -x php
```


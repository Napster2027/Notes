### #  `Using ffuf for File Read(LFI)` -

```bash
ffuf -u http://snoopy.htb/download?file=FUZZ -w /opt/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt -mc all -ac
```

`-mc all` to allow all codes, and `-ac` to let it smart decide what to filter.

### # `Subdomain Enumeration` -

```bash
ffuf -u http://10.10.11.193 -H "Host: FUZZ.mentorquotes.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 18 -mc all
```

`-mc all` to get all codes.

```bash
ffuf -u http://10.10.11.212 -H "Host: FUZZ.snoopy.htb" -w /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -mc all -ac
```

`-mc all` to show all status codes and `-ac` to allow for smart filtering of the common response.

### # `Content Discovery-Recursion` -

This switch tells ffuf that if it enounters a directory it should start another scan within that directory and so on until no more results are found.

```bash
ffuf -w ~/wordlists/common.txt -recursion -u http://ffuf.me/cd/recursion/FUZZ
```

### # `Content Discovery-File Extensions` -

Use the below scan with the `-e` switch to specify the extension type to add onto the end of every word in the wordlist to find the correct log :

```bash
ffuf -w ~/wordlists/common.txt -e .log -u http://ffuf.me/cd/ext/logs/FUZZ
```

### # `Content Discovery-Param Mining` -

Using the below request we can try and find the missing parameter :

```bash
ffuf -w ~/wordlists/parameters.txt -u http://ffuf.me/cd/param/data?FUZZ=1
```

### # `Content Discovery-Rate Limited` -

We're using the `-mc` switch to only display http statuses 200 and 429 :

```bash
ffuf -w ~/wordlists/common.txt -u http://ffuf.test/cd/rate/FUZZ -mc 200,429
```

Now try running the below command, the -p switch causes the application to pause 0.1 seconds per request and the -t switch creates 5 versions of ffuf which means a maximum of 50 requests per second :

```bash
ffuf -w ~/wordlists/common.txt -t 5 -p 0.1 -u http://ffuf.test/cd/rate/FUZZ -mc 200,429
```

### # `Authenticated Discovery` -

```bash
ffuf -u http://drive.htb/FUZZ/getGroupDetail/ -w <(seq 1 500) -fc 500 -H "Cookie: csrftoken=GWHHBpfjentV8FG7IVYiKgMAmK5wNVaF; sessionid=c8xebin9cekvgy59r1de8wvfmllxgrnu"
```

- In above `ffuf` hit `http://drive.htb/FUZZ/getGroupDetail/` to check for all group numbers.
- For the wordlist, I have used `-w <(seq 1 500)`, which uses [process substitution](https://en.wikipedia.org/wiki/Process_substitution) to pretend there’s a file containing the numbers 1 through 500 one per line. 
- `-fc 500` will hide results that return HTTP 500, which is what happens when there’s a non-existent id. You’ll also need to include your cookie, which you can grab from Burp.


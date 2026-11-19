System: Linux
Difficulty: Very Easy

IP Address: 10.129.1.127

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.1.127 -sC -sV
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-title: Did not follow redirect to http://ignition.htb/
|_http-server-header: nginx/1.14.2
```

The request is visualized in the browser.
However, it only throws a link to a specific domain. This line is entered through the `/etc/hosts` file.
```
10.129.1.127	ignition.htb
```

Now the website has been returned correctly, there is nothing much interesting on it.
Directories are enumerated on the web server.
```
kali@kali:~$ dirsearch -u http://ignition.htb
```

An interesting login form has been found. It is now accessed through this link.
```
http://ignition.htb/admin/login
```

Multiple logins are tried from common passwords through the user `admin`. The password is `qwerty123`.

After successfully logging in, the flag is now displayed in the browser.

The flag contents is in below.
```
797d6c988d9dc5865e010b9410f247e0
```
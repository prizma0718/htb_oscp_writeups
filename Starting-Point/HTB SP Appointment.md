System: Linux
Difficulty: Very Easy

IP Address: 10.129.111.30
# System Flag

Ports are scanned through NMAP.
```
kali@kali:~$ sudo nmap 10.129.111.30 -sC -sV -p- -v
```

An HTTP Port has been found on the target.
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-favicon: Unknown favicon MD5: 7D4140C76BF7648531683BFA4F7F8C22
|_http-server-header: Apache/2.4.38 (Debian)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Login
```

The contents are explored on the browser. 
![[appointment_1.png]]

The login form is most probably vulnerable from SQL Injection, based on this pattern.
The typical payload `admin' OR 1=1#` is used with a random password to test the exploit.

The contents of the flag are below.
```
e3d0796d002a446c0e622226f42e9672
```
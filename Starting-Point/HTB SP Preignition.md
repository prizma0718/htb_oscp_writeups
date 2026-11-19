System: Linux
Difficulty: Very Easy

IP Address: 10.129.109.197

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.109.197 -p- -v
```

An HTTP Port was returned from the command.
```
<SNIP>
PORT   STATE SERVICE
80/tcp open  http
```

The webpage contents is requested.
```
curl http://10.129.109.197
```

Here are the source code returned.
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

Directory Enumeration will be attempted.
```
kali@kali:~$ dirsearch http://10.129.109.197
```

```
Target: http://10.129.109.197/

[14:33:55] Starting:
[14:34:00] 200 -  999B  - /admin.php
```

A page named `admin.php` has been found. This page is now visited.
![[preignition_1.png]]

The page is asking about credentials. The default credentials of `admin:admin` are used as a test.

Those credentials were valid, returning the flag value with success.
The contents of the flag are below.
```
6483bee07c1c1d57f14e5b0717503c73
```

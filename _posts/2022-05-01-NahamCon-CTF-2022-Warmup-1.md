---
title: NahamCon CTF 2022 - Warmup Writeup Part 1
published: true
---

Over the last few days I've taken part in [NahamCon CTF 2022](https://ctf.nahamcon.com/), coming in **233**/4034 teams in the end which I'm pretty happy with.  I'm writing up all my solves, with this post covering the first 4 warmup challenges.

# FlagCat

> Do you know what the cat command does in the Linux command-line?
> 
> Attachment: flagcat

Sounds simple enough, let's see what's going on with the file.  First we download it:

```
wget -O flagcat https://ctf.nahamcon.com/files/fee3ca803ce20c9fad38316e749cafe5/flagcat?token=<my-token>
```

Lets see what's in it with `cat flagcat` and boom, there's our first flag:

```
 ---------------------------------------- 
| flag{ab3cbaf45def9056dbfad706d597fb53} |
 ----------------------------------------
        ||
 (\__/) ||
 (•ㅅ•) //
 / 　 づ

```

# Quirky

> This file is seems to have some strange pattern...
> 
> Attachment: quirky

First we download the file:
```
wget -O quirky https://ctf.nahamcon.com/files/5acb7200ca497cad17247fe2c9a45bdc/quirky?token=<my-token>
```
Opening it up shows a byte string:
```
\x89\x50\x4e\x47\x0d\x0a\x1a\x0a\x00\x00\x00\x0d\x49\x48.......
```
Let's paste that into [CyberChef](https://cyberchef.org/) and use "From Hex" to see what we have:

![cyber chef screenshot]({{ site.url }}/assets/post-images/nahamcon-warmup/quirky-hex-decode.png)

OK so it's a PNG file.  We can use the following python to convert it back (there's probably also a huge number of commands and tools people will scream at me for not using, what can I say, I'm a Software Engineer at heart - why spend 3 seconds using a tool when you can spend 3 minutes writing a script.....):

```python
data = b'\x89\x50\x4e\x47\x0d\x0a............' # truncated for brevity
fo = open("quirky.png", "wb")
fo.write(data)
```

Opening the PNG:

![cyber chef screenshot]({{ site.url }}/assets/post-images/nahamcon-warmup/quirky.png)

That's a QR code, lets scan it with [QR Scanner](https://www.qrscanner.org/scan-qr-code-from-image) to get the flag

![cyber chef screenshot]({{ site.url }}/assets/post-images/nahamcon-warmup/quirky-qr.png)

Boom! Another flag down

# Prisoner

> Have you ever broken out of jail? Maybe it is easier than you think!
> 
> Connect with:
> 
> \# Password is "userpass"
> 
> ssh -p 31650 user@challenge.nahamcon.com

Sounds good, let's connect:

```
_________________________
     ||   ||     ||   ||
     ||   ||, , ,||   ||
     ||  (||/|/(\||/  ||
     ||  ||| _'_`|||  ||
     ||   || o o ||   ||
     ||  (||  - `||)  ||
     ||   ||  =  ||   ||
     ||   ||\___/||   ||
     ||___||) , (||___||
    /||---||-\_/-||---||\
   / ||--_||_____||_--|| \
  (_(||)-| SP1337 |-(||)_)
          --------

Hello prisoner, welcome to jail.
Don't get any ideas, there is no easy way out!
: 

```

Typing doesn't get any response, so let's try a few ways to interrupt the app...
- CTRL-C - nothing
- CTRL-Z - nothing
- CTRL-D - Aha!

OK so at this point we've dropped into a python REPL

```
    ||   ||     ||   ||
     ||   ||, , ,||   ||
     ||  (||/|/(\||/  ||
     ||  ||| _'_`|||  ||
     ||   || o o ||   ||
     ||  (||  - `||)  ||
     ||   ||  =  ||   ||
     ||   ||\___/||   ||
     ||___||) , (||___||
    /||---||-\_/-||---||\
   / ||--_||_____||_--|| \
  (_(||)-| SP1337 |-(||)_)
          --------

Hello prisoner, welcome to jail.
Don't get any ideas, there is no easy way out!
: Traceback (most recent call last):
  File "/home/user/jail.py", line 27, in <module>
    input(": ")
EOFError
>>> 

```

Nice, from here we can import the `os` package and see what files are in this folder:

```
>>> import os
>>> os.listdir(".")
['flag.txt', 'jail.py', '.user-entrypoint.sh', '.profile', '.bashrc']
>>> 

```

Well well well, if it isn't our old friend `flag.txt`.  Time to grab the contents:

```
>>> open("./flag.txt", "r").read()
'flag{c31e05a24493a202fad0d1a827103642}\n'
```

And that's more points on the board

# Exit Vim

> Ah yes, a bad joke as old as time... can you exit vim?
> 
> Connect with:
> 
> \# Password is "userpass"
> 
> ssh -p 30415 user@challenge.nahamcon.com

A classic.  Connecting we go straight into a vim instance.  Typing `:q` to quit and we get a flag.

```bash
flag{ccf443b43322be5659150eac8bb2a18c}
Connection to challenge.nahamcon.com closed.
```

You can find part 2 [here](https://ruddles.github.io/NahamCon-CTF-2022-Warmup-2)
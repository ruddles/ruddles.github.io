---
title: NahamCon CTF 2022 - Warmup Writeup
published: true
---

Over the last few days I've taken part in [NahamCon CTF 2022](https://ctf.nahamcon.com/), coming in 233/4034 teams in the end which I'm pretty happy with.  I'm writing up all my solves, with this post covering the warmup challenges.

# Finding the final vulnerability

At this point I've had a lot of time to dig around in the front end and can't think of anything else to try there, so let's have a dig in the database.

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
wget https://ctf.nahamcon.com/files/5acb7200ca497cad17247fe2c9a45bdc/quirky?token=<my-token>
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

![cyber chef screenshot]({{ site.url }}/assets/post-images/nahamcon-warmup/quirky.png)

That's a QR code, lets scan it with [QR Scanner](https://www.qrscanner.org/scan-qr-code-from-image) to get the flag

![cyber chef screenshot]({{ site.url }}/assets/post-images/nahamcon-warmup/quirky-qr.png)
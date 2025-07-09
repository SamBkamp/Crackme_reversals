# Pico Format String 2

from [here](https://play.picoctf.org/practice/challenge/448?category=6&page=1)

## Write up

This was the first format string challenge from pico ctf that challenged me, and challenge me it did.

When I first opened the source file I was confused, I see the format string vulnerability but no gets() or other unsafe write call. How was I supposed to write data to stack like the challenge description suggested? I quickly googled if there was a way to write with printf, my thought was that maybe you could change the output stream from stdout to a file or something. I was close! Turns out there exists a %n specifier that doesn't print anything but merely writes the amount of bytes printed thus far in this printf call to a `int *` location vararg.

So from this point I thought it was simple, I could write numbers to any location in registers or on the stack. Easy. But to pass the challenge I had to write to the location 0x404060 (the location of the sus variable which I found on my first time running the binary with gdb). This is where I ran into my first problem, I couldn't find that location on the stack anywhere. I wasn't sure how to write to that location if I don't have the location at all. First, I tried to overwrite a nearby location, 0x403e17 or something in the hopes it would overflow to 0x404060 but I just got a segfault. I kept leaking stack addresses in the hopes I could find that variable on the stack but I couldn't simply because the variable was used only once (before the cmp) and was loaded directly from program memory (0x404060) into a register.

I got stuck and wanted to know the next step so I found a writeup on this challenge to see what they did next without spoling myself too much. I used [this](https://blog.thecyberthesis.com/blog/writeups/picoCTF/pwn/format-string-2) write up to guide me. I only had to read a few sentences before I came across:

> We could also send the addresses as input to the program which would put those addresses on the stack.

Of course!!! My input gets stored on the stack, so all I'd have to do is pop addresses until we got to the end of my input where I put in my address. My exploit payload at this point looked something like this:

```python
payload = (b"%p|"*24) + "%n|" + p64("0x404060")
```

and this worked, I managed to overwrite the sus location variable!

Now all I had to do was print a bunch of characters until the number was 0x67616c66...

Unfortunately I quickly found out that number was way too big. It was also impossible to overwrite the right hand argument of the cmp as it was hardcoded into the cmp assembly instruction so yet again I was stuck and yet again I turned to the write up previously mentioned for help. Before reading the solution (I was still keen on solving this puzzle with as little help as possible), I saw a link to a seperate blog post about format string write vulnerabilities in general. It talked about generating code such that you could write arbitrary bites to an address using the `%hh` length modifier in printf. See, normally %n would write a full 8 byte integer to whatever location youre writing too, but using the length modifier `hh` you could print one singular byte only. If you combine this with putting several different addresses on the stack (offsets of a 3 byte address for example) you could write whatever you wanted*.

What I took this to mean is that I needed to write to 0x404060, 0x404061, 0x404062 and 0x404063 with 0x67, 0x61, 0x6c and 0x66 respectively. So while the whole number 0x67616c66 was way too large to print and then write with %n, 0x60-0x6f was much smaller and much easier.

The other thing thats important to mention is that %n prints the amount of bytes printed to stdout in this function call already, which means it can only go upwards. This means I had to write to addresses in ascending order of the solution bytes. Ie: 0x404062, 0x404063, 0x404060 then 0x404061 so that I could write 0x61, 0x66, 0x67 then 0x6c respectively (ascending order).

Once I figured this out, the rest was relatively straight forward, I managed to write the first byte (0x61) to 0x404062, then print a character of width 5 with `%5c` in the printf. It looked something like this:

```python
payload = (b"%p"*19) + "%hhn" #padding + first write to first address
payload += "%5c" #print an extra 5 characters to stdout to increase the value of %n
payload += "%hhn" #write to second address
payload += "AAA" #padding (idr what it was exactly at this stage of the solving but its irellevant anyway)
payload += p64("0x404062") #offset 1
payload += p64("0x404060") #offset 2
payload += p64("0x404063") #offset 3
payload += p64("0x404061") #offset 4
```

If you have a keen eye, you'll see that something is wrong here. Namely, the order of the offset addresses im writing too. I didn't understand it at this point, instead I thought I had gotten the offsets reversed (the address starts from the back going forward) but that wasn't the case. The reason why this code works is because the "%5c" format 'takes up' another argument before the next write, so offset 2 was printed as a 5 width character and offset 3 was written to instead. I didn't notice this as I had only been focussing on writing the first 2 bytes and ignored the last 2 for now.

After this worked I thought I was only a minute or so away from solving the challenge when I tripped over the last stumbling block. I still hadn't realised that the %5c format was using one of my addresses of the stack, so when I tried to write to the first 3 addresses, I for some reason could never seem to be able to get all 3 addresses aligned properly. The middle address kept disappearing!

I went back to the blog post in question to see if I was missing anything, and it was onl after looking up what the $ format meant did I realise what happened and how to get around it. The $ format allows you to specify which argument should be converted with that format. eg:

```c
printf("Here is a string: %s here is a character: %1$c", someString);
```

Although there are 2 formats, I am only using 1 variable, this is because the second format is directing the conversion to be done on the 1st vararg, and not on a second one as it would be by default.

We can use this same format to use the address both to pad the amount of bytes written and to write to the address itself without having to faff around with extra padding addresses specifically for the %5c directive.

Once I figured that out, I solved it!

*pwned*
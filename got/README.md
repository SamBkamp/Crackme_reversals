# GOT - PwnMeCTF2025 Qualifiers

I found this challenge on the [pwnmectf2025](https://github.com/Phreaks-2600/PwnMeCTF-2025-quals/tree/main/Pwn)

### write up

the first thing I noticed when starting this challenge is that there was no source code, I expected this to make it much harder but really it didn't pose much of a hurdle.

A cursory glance at the assembly in main showed these function calls:

- puts
- puts
- fwrite
- scanf
- puts
- puts
- fwrite
- read
- puts

Now the very first thing that stood out was the glaring `read()` call - it will write wherever for as many characters as you want.
Looking at the code some more we also see there is a function called `shell()`, a clear target to jump to.

Its also worth noting that this was compiled without a canary and also only partial RELRO.

when executing the code it asks us two questions:

```
Hey ! I've never seen Game of Thrones and i think i misspelled a name, can you help me ?
Which name is misspelled ?
1. John
2. Daenarys
3. Bran
4. Arya
```

then
```
>
Oh really ? What's the correct spelling ?
```

finally followed by
```
>
Thanks for the help, next time i'll give you a shell, i already prepared it :)
```
looking in the code we can see that there is a call to scanf for the first input, this is then checked to see if its less than 4. After this the `read()` reads the second input into the location 0x4040a0. This confused me at first, what was at that location that was so interesting? gdb showed it was in a section called PNJ, I couldn't find anything about this online so I was stumped and went back to look at the first input. I noticed the symbol for this address was referenced later on in the program. I noticed it was shifted left by spaces and gets added to the number 0x404080... interesting. This address then gets passed to the read() function as the location where our input is stored. bingo! We are basically given free reign to write arbitrary data to arbitrary addresses, i was close.

Unfortunately there is one caveat, the second number is checked to see if its not less than 4 and it will exit the program should it be greater than 3. So we can only type in 1-3, meaning an offset of 0x20 to 0x80 off of 0x404080. I began poking around there to see if I could find anything at those addresses but there wasn't anything interesting, mostly zeroes. I was confused for a bit until I realised the check ony checks if its less than 4, which means negative values are valid and we can go backwards!

I this gave me a lot more freedom where to write my exploit to, but where would I write it too? We are also partially constrained by the fact our number is shifted left by 5, meaning we can only write to addresses in multiples of 0x20. I was trying to find random locations to write to that would be executed later on in the code, i even tried to write to the area in memory where the binary itself was stored. Unfortunately none of the addresses lined up cleanly. I kept looking around for addresses to write to for quite a long time. Eventually I decided to simply list all the addresses that ended in multiples of 0x20

`objdump -D got -Mintel | grep 4040[0-9a-f]0`

and I got this:
```
sam@fish-mobile ~/Crackme_reversals/got $ objdump -D got -Mintel | grep 4040[0-9a-f]0
  401030:	ff 25 ca 2f 00 00    	jmp    QWORD PTR [rip+0x2fca]        # 404000 <_exit@GLIBC_2.2.5>
  401050:	ff 25 ba 2f 00 00    	jmp    QWORD PTR [rip+0x2fba]        # 404010 <__stack_chk_fail@GLIBC_2.4>
  401070:	ff 25 aa 2f 00 00    	jmp    QWORD PTR [rip+0x2faa]        # 404020 <read@GLIBC_2.2.5>
  401090:	ff 25 9a 2f 00 00    	jmp    QWORD PTR [rip+0x2f9a]        # 404030 <__isoc99_scanf@GLIBC_2.7>
  4010f0:	b8 50 40 40 00       	mov    eax,0x404050
  4010f5:	48 3d 50 40 40 00    	cmp    rax,0x404050
  401109:	bf 50 40 40 00       	mov    edi,0x404050
  401120:	be 50 40 40 00       	mov    esi,0x404050
  401125:	48 81 ee 50 40 40 00 	sub    rsi,0x404050
  40114b:	bf 50 40 40 00       	mov    edi,0x404050
  401164:	80 3d f5 2e 00 00 00 	cmp    BYTE PTR [rip+0x2ef5],0x0        # 404060 <__bss_start>
  401176:	c6 05 e3 2e 00 00 01 	mov    BYTE PTR [rip+0x2ee3],0x1        # 404060 <__bss_start>
  401272:	48 c7 c2 80 40 40 00 	mov    rdx,0x404080
  404010:	56                   	push   rsi
  404020:	76 10                	jbe    404032 <_GLOBAL_OFFSET_TABLE_+0x4a>
0000000000404040 <__data_start>:

```
the first thing that caught my eye was the first 4 jumps. I could write to those and make them call our shell function right? The problem is that this area of memory wouldn't execute in the normal flow of the program and I couldn't hijack the stack nor rip. I kept looking and I felt quire stumped until I googled what a global offset table was (the last address in this dump) and it hit me like a tonne of bricks. Global offset table... GOT... How did i miss this???????

After a quick google I learned more about what a global offset table was, I had a basic understanding - something to do with libc functions, but now I knew exactly how it worked (minus ld, thats a black box to me and seemingly everyone). This is where I recalled that this binary was compiled with only partial RELRO, meaning that while .got was non writable, .got.plt was. The global offset table is a section in a dynamically linked binary that deals with calls to libc, both running the code that calls the linker and running code that stores cached function calls, basically the first time a libc function is called, it needs to be linked by the linker but subsequent times it can be called directly as we already have the address of the function and its not going to change half way through a program. So after a libc function is called once, the address gets stored in the global offset table and this is what we could overwrite, so that the address would instead point to our shell function.

Now another problem I had is that 0x404020 pointed to `_exit()`, a function that wasn't called anymore in the flow of the program. What would the point be in overwriting that? I needed to overwrite 0x404028 - the location of puts. Clearly at this point I had fried my brain with this challenge because it took me a second to realise I could just write 16 bytes instead of just 8. I could overwrite the _exit() location with garbage data so I could get to the puts function and overwrite that with the location of my shell function. This worked! I managed to run the shell function and get a shell.

**pwned**


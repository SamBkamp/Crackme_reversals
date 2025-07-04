# simple PWNME from Liveoverflow

I found this (here)[https://github.com/LiveOverflow/pwn_docker_example/tree/master/challenge]


## Write Up

Because this is a pwn challenge, we have access to the source code. In the source code the first thing that we can see is that this binary was compiled with `-fno-stack-protector` and `-no-pie`, meaning that this code will always run at the same addresses and without a canary.

Looking further down we can see a `gets()` call and a suspiciously name `backdoor()` function. This points towards a clear bufferoverflow and ret2win exploit.

Now, I originally thought that this exploit was about overwriting the string inside the `system()` call in `remote_system_health_check()`. I thought I'd have to overflow the area in memory where that string was. Of course in hind sight this was a little silly as the string would clealry be stored in the .data section of the binary and not the code section so it would be impossible to overwrite that string without overwriting the rest of the program.

After messing around with that for a while, I noticed that I could overwrite the base pointer rbp, and by extension also the address one behind it, the one that would be popped into the rip instruction pointer after a ret instruction.

Another important bit to note here is that I had trouble reaching the ret instruction as gdb would fork into the system call. I had to use the gdb setting `set follow-fork-mode parent` to make sure I stayed in our mainline function.

After all this (which took longer than I'd like to admit), I used the cyclic function in pwntools to figure ut where the base pointer and the 8 bytes behind it were. After overwriting that location with the location of the backdoor function I managed to return into the backdoor function, done!

Or so I thought, we still segfaulted because the stack pointer was not aligned on a 16 byte boundary. This was simple however, as all I had to do was find the location of another ret to jump to first. This means overwriting the first address with a ret and then after that the address of the backdoor such that the first ret pops the stack (leaving it at the 8 byte boundary) which gets returned to another ret, which again pops the stack (thus aligning it to 16 bytes) and then jumping to the backdoor function.

Using `objdump -D system_health_check` I was able to disassemble the whole file and find a ret that I liked. Because the whole binary was not position independant, I could just put that address directly onto the stack. This then worked and I got access to a shell!

*pwned!*

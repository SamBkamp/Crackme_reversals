# a very **very** simple format string vulnerability

I got this from [here](https://ctf101.org/binary-exploitation/what-is-a-format-string-vulnerability/) to get a basic understanding of format string vulnerabilites without the extra faff of solving a ctf.

## Write up

So we had the source code here and we could see the glaring format string vulnerability where our input is echoed back to us directly from inside a printf.  We could also see that our secret was stored as an integer so all we had to do was keep printing hex integers with %p. After printing 7 %p's we could see our secret leaked. Okay so this was easy to exploit, but one thing I didn't understand at first was why it was 7 %p's we had to print.

On first glance I thought we just had to print 4, as our buffer address was pointing to 0x50 and our secret was at 0x54. But this was not the case. Some forums suggested that printf reads the varargs from the stack, but thats not entirely true either as we'd only have to print 2 %ps to get to our secret, so what gives? Well it turns out printf doesn't read the varargs from the buffer being referenced nor the stack (directly) but a much simpler answer; It just reads the varargs in normal c sysv calling convention. I don't know why this took me a few minutes to realise when in hindsight its obvious. When I call printf in c code it might look like:

```c
printf("this is some pointer %p and heres a string %s\n", somePointer, someString);
```

Where I just pass everything as arguments to the function, of course all the arguments would be passed with normal calling convention! So, why is it 7? Well we have the first 6 arguments passed via registers (RDI, RSI, RDX, RCX, R8 and R9) and only *then* does it start to read from the stack. 6 arguments from registers (which doesn't contain our secret) + 2 arguments from the stack (the second of which does contain our secret) gives us 8 arguments. But of course the first argument (in RDI) is already passed for us (its our string, ie the format part of printf) by the call to printf, that means we just need to call 7 on top of that for a total of 8 arguments passed to the function. And there we have it!

While this wasn't a particularly hard challenge at all, it did give me insight specifically into when printf actually starts reading from the stack, namely only after the first 6 (1 format + 5 vargargs) arguments. 
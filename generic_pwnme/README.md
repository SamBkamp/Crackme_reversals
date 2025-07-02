# pwn me - a random one I found online

I found this random pwnme code online and I wanted to give it a try to refresh my pwndbg and pwntools skills

## Write up

This binary used a call to `read()` to read input from std in, this would be stored in the variable `name`. Then, another variable `secret` was compared to the number 0x1337, but how is it possible to pwn this if we can't change impact the variables being compared? Well, there is a small bug in the code that makes it possible. `name` has 100 bytes of space (100 chars, each char is 1 bye) but the read() function reads 0x100 bytes, which is 256 bytes. This means the read function can read more bytes from stdin as there is space to store it, this allows us to overflow the name buffer on the stack and potentially smash the stack and change the secret variable.

To find out the offset of where the secret variable is in memory I used the cylic function in pwntools and pdbg.

```python
p.sendline(cyclic(256))

```

this allows me to look into the memory area and find out the offset using this algorithm. The program compared 0x62616163. Putting this into `cyclic -n 4 -l baac` gave us an offset of 204. After sending 204 A's and then 4 B's to sanity check:

```python
p.sendline("A"*204 + "B"*4)

```

I saw that the memory area where `secret` was, was still being overwritten by a bunch of A's. I have no idea why, I assume I used cyclic wrong. Instead, I did `x/s $rbp-4` (which was the location in memory where secret was stored) and gdb told me there were 96 A's and then 4 B's. Meaning we had 96 As too many.	So I simply subtracted 96 from 204 to get our final number of 108 A's of offset. Sanity checking this again with:

```python
p.sendline("A"*108 + "B"*4)

```

Resulted in the `secret` area of memory to be completely overwritten with Bs. So, now I had the location and offset and all I had to do was replace the B's with the correct value, namely 0x1337.

**pwned!**

*note: I compiled this with -fno-stack-protector to remove the canary*
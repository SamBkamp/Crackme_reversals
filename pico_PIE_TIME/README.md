# PIE TIME from picoCTF
a Write-up

## solution

This challenge was a fun one, the title gave away it was something to do with position independant code, but running the executable made it clear what the deal was. When running the program you are greeted with you get:

```bash
Address of main: 0x55fb0856033d
Enter the address to jump to, ex => 0x12345:
```
It gives you the address of main, then allows you to jump anywhere arbitrarily given an address. Looking at the source code shows us that we're looking to jump to a function called `win`, which should be easy enough to find.

First off, we get the address of main from the program and store it - we'll need it later. Then I used GDB to find the address of the win function for a given program running:

`$1 = {<text variable, no debug info>} 0x5555555552a7 <win>`

and this program gives us a main address of: 0x55555555533d.

In my exploit.py program I created an offset variable that was the latter subtracted by the former to get the difference in locations between the two functions, given these are from the same run through of the program it should give us a valid offset and this offset should hold for any execution of this program.

Next, I took the leaked main address the program gave us and subtracted the offset from it, giving us the address of win for this instance of the program. Formatting it as a hex string and sending it to the program jumped us to the win function and we got our flag!

*pwned*
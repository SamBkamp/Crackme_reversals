# Crack me - ["Password Login"](https://crackmes.one/crackme/5c90a72d33c5d4776a837f07)

## Write up

I first started with a simple `file` command, this was the output:
```
crackme: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=7f9bfa4851bb1a704fefd80161ceb168769535c8, not stripped
```

So far, looks easy.

Running the program gives you this:

`Enter the password:`

putting in a random password gets you "login failed". Lets throw this through radare2.

After typing `aaa` then `s sym.main` we can see that the program calls this function `checkPassword()` before making the final jump. So lets go there.

`is | grep checkPassword`

`s 0x1275`

`VV`

Now we are in this function. We can see that this function gets passed a CPP String object. We can also see that there is a call to `length()` before a conditional jump. We can see this compares the length of our string and sends us to 0x13c8 if we don't match the length and returns us from the program. A quick peak in gdb sees that its comparing us to the value 0x7, so our string has to be at least 7 characters to pass this check.

Next, we are confronted with a for loop that loops until 3. Here, it subtracts 1 from 3 and compares it to the value of i. I imagine it looks like this:

```c
for(int i = 0; i < 3; i++){
	if(i == 3-1){
	     //some code
	}	
}
```

In this loop, some changes are performed, presumably on our inputted string.

After this loop ends, we head to the main check of the program where it seems to call begin() and end(). At this point I struggled to find my input string in the code. I went back to the main function and tried to track down its life there. I am not sure if this is compiler optimisation, intentional obfuscation or just how cpp/gpp deals with strings but I was unable to find where in memory my string even was. The program seemed to call both the << operator and getline() in the main function. I wrote my own cpp program to disassemble and take a closer look.

After compiling and looking at the assembly code it hit me like a train, I was treating my string  like a c string (a null-terminated byte array). It's no wonder I couldn't find the text I was inputting as its in a class and not in memory directly like it might be in C. an extremely silly mistake.

Using `p (char *)$rbp-0x30+0x10`, we can see the string at that address. We get -0x30 from the offset pointed to by the assembler in the main function and +0x10 is the offset where the raw characters are actually stored (I found this out by examining the compiled cpp code I wrote with debugging symbols). We can see that our string in the main function is stored at $rbp-0x30 and $rbp-0x50

Now that we have a way to see what happens to our string, we can go back to the checkPassword function and see how its manipulated in (im guessing) the for loop above.

printing the std::string at address $rbp-0x60 (`print (char *)$rbp-0x50`) regularly throughout this loop sees us start with the string "dec" -> "dek" -> "dekcar" -> "dekcarc". Of course this on the stack so its stored in reverse.

Trying the key "cracked" then gives us the prompt

`Login successful`

*Solved!*

### closing thoughts:

this crack me was really rather straight forward, more an excersize in obfuscation more than anything else. The main thing I learned from here of course is examining cpp std::string objects, that little mistake took up maybe 80% of my solving time. 


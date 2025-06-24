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

Now we are in this function. We can see that this function gets passed a CPP String obeject. We can also see that there is a call to `length()` before a conditional jump. We can see this compares the length of our string and sends us to 0x13c8 if we don't match the length and returns us from the program. A quick peak in gdb sees that its comparing us to the value 0x7, so our string has to be at least 7 characters to pass this check.

Next, we are confronted with a for loop that loops until 3. 
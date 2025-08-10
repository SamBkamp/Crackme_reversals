# picoCTF Even RSA can be broken??
a write up

## Solution

I first got a bit confused that the supplied source code wasn't running, turns out the authors left out some of the code to obfuscate things like the get_primes() function. After figuring this out I just connected to the server to see what we get.

After connecting you are given this:
```
N: 17078350002645953979610384475146752383811214305902672199308054922366642494852419271607691702569460810501954600975037733591402136305154154474667516557842018
e: 65537
cyphertext: 7796760919085948832399137357708002287004950681257661944944713053317254839990079395656897823717435929711152713363772987762568738782270217575210927778887879
```
So, they give you an N, and e and a cypher text. This **should** be cryptographically secure, but seeing that this is a CTF, it probably isn't.

The first thing I considered is that this was a case where `p` and `q` were too close together and you could fermat factor it out. After looking at the numbers though, I noticed that N ended in an '8'. I made a few more requests and every single time N was coming out as even. Ah ha!! I have solved the bulk of the challenged.

Because `n = p*q`, `p` and `q` have to be prime. We can be certain that p has to be 2 and q has to be half of n (or vice versa). Now we have figured out p and n which is the bulk of the work.

The rest of the time I spent getting owned by modulo arithmetic. Once I wrote code to calculate thetaN (which really is just p-1) I then calculated d. I could've used pow() to do this (as is commented on line 36) but I wanted to kinda get how it worked. Then came the really annoying part, doing the actual decryption. I first started with the obvious `(c**d)%n` but that took forever, then I read about the square and multiply approach on the wikipedia article on how this is calculated quickly, but even after implementing this it took forever. I then saw what the source code did where they used the `pow()` function with three arguments. It turns out the `pow()` function can take a modulo as an argument and use this kind of expression to really really really quickly calculate the result using some extremely weird modulo arithemetic tricks. I spent a lot of time (had dinner) while waiting for my other implementations to work. I also used the long_to_bytes function from pycrypto to turn it into a byte string. I'm sure theres ways to make it work without pycrypto but whatever.

*solved!*
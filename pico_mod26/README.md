# mod26 from picoCtf


## solution

so the caption of the challenge kinda gave the game away (and to be honest so did the title) that this was about rot13 and the ceaser cypher to decode the flag string.

I wrote a quick and dirty python script that would spit out the flag (mostly). I managed to solve this challenge under 20 minutes and it was mostly ironing out the weird bits of my python code.

#### the code

I started by iterating through each character in the string and the first thing I did was to ignore the characters that didn't need rotating (like the _ and {} characters). After this I started creating an offset to get from the character processed to its corresponding nth value in the alphabet (a=0, b=1, c=2, etc.) At first I did this with a sketchy little brute force but then I realised an easier way was to just do the funky little ascii math instead. in ascii, the characters are all in alphabetical order starting from 0x41 and 0x61 for capitals and lowercase respectively.

So, to get this nth value I just had to subtract the ascii code of each character with either the lowercase starting point or the uppercase starting point. Eg: B = 0x42. 0x42-0x41 = 1 therefore b = 1. The offset it would be subtracted by depends on if its capitalised or not, hence the ternary operator in the offset variable declaration.

Lasty comes a oneliner that would be a code QA manager cry, so lets break it up into each operation:

- first, we convert the character to its ascii equivalent (which we have to do in python) using the ord() function
- then, we take that number and do the subtraction mentioned above by the offset (ord(thing[i]) - offset)
- next, we add our rotation (13) to the number with +m
- then we mod it by the length of our list so the number we get "rolls over" should it go < 0 or > 26 and wraps back around to the start or end.
- lastly, we use that nth offset after the rotation to get back to a character from our alphabet list with Ldict[] or lDictCaps[].


I'm sure I could have wrote this code to use one less if statement but whatever.

This gives us the key `picoCTF{next_time_INll_try_2_rounds_of_rot13_TLcKBUdK}` and using my big brain I can assume the `N` `INll` should probably be a `'`, so I replaced it and succesfully submitted it. I wish there wasn't a `'` in the flag though so that us solvers could be sure of the character-space we should be using (this kinda goes for the numbers too, I didn't clock that they shouldn't be rotated pretty late in the process).

Anyway, solved!
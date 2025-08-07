# HASHCRACK
from picoctf

## solution
So this was not my first time bruting hashes and using tools like hashcat, so after identifying the hash as md5 I simply ran md5 in brute force mode for md5 with:
```bash
hashcat -a 3 -m 0 482c811da5d5b4bc6d497ffa98491e38
```
However after reaching 9 characters in like 8 minutes (which is quite fast admittedly) I figured a brute force attack probably wasn't the way to go. I then found an online source for the iconic rockyou list from 2009 to run a dictionary attack on the hashes. This source had many dictionaries but I wasn't super keen on downloading 3gb text files and figured rockyou would do, especially for an easy challenge. I kept up this strategy for each hash:
```bash
hashcat -a 0 -m 0 482c811da5d5b4bc6d497ffa98491e38 rockyou.txt
hashcat -a 0 -m 100 b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3 rockyou.txt
hashcat -a 0 -m 1470 916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745 rockyou.txt
```

the rockyou list served me well and cracked each of these hashes very quickly!
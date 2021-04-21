# Viginere
![](https://img.shields.io/badge/category-crypto-blue)

## Description
If you're stuck, you might simply need to SHIFT your thinking a bit…

## Solution
The encrypted.txt the problem gives you contains an encrypted message. Bruteforcing the Caesar cipher gives us

```
Some website said that leaving your clothes DAMP for **12** hours after washing it will cause it to mildew. However, if that does happen, the solution is simply washing it again! Oh, right, I was supposed to encrypt a flag...

HVT{R_Q0PU_I04I}
```

The flag is still encrypted, but the message isn't. Notice that DAMP and 12 are emphasized (and that the title of the problem is Viginere. Trying to decrypt `HVT{R_Q0PU_I04I}` using viginere and the key DAMP doesn't give anything. However, if we shift DAMP by 12 letters, we get ROAD. Decoding `HVT{R_Q0PU_I04I}` with the key ROAD gives us our flag: `CTF{A_L0NG_R04D}`

# Kevin Needs Help, Again
![](https://img.shields.io/badge/category-crypto-blue)

> My friend Kevin accidentally encrypted his flag and lost his private key, and needs your help! Luckily he’s new to encryption, so there might be a weakness in his algorithm, can you find it?

Files: rsa.py

<details>
<summary>Hint:</summary>
Being a competitive programmer, Kevin only knew how to generate primes in O(nloglogn), and Kevin does not have much time to create this key meaning he can only run his prime generation in a short amount of time.
</details>


Kevin Needs Help Again is my sequel to another problem named [Kevin Needs Help](https://dmoj.ca/problem/kevin).

Let's take a look at the file

```python
from Crypto.Util.number import *

##################################################################################################################################
#																																 #
# No way you can brute force this right? It would take at least O(2^79) to brute force. It's absolutely secure, right? right?... #
#																																 #
##################################################################################################################################

N = 1
e = 65537

flag = b"REDACTED_NO_FLAG_4_U"
flag = bytes_to_long(flag)

for _ in range(10):
	p = getPrime(80) # Generate random 80 bit prime
	N *= p
	print(p)

c=pow(flag,e,N)

print(f"N = {N}")
print(f"c = {c}")
print(f"e = {e}")

# N = 277374920610630854709341081430070381895040367159247831346063113357346996861997096842985380163195142293041723827605803582959095385096852674868844481299485182299736902846354208891395068488674159312782402962289252130974324110540905208754122079
# c = 227688146150768027567515600674768662657437530439630813908158076963410580173896425412413514242614358728441978777344374383817175868212259820714403179898822987968118941373698837582673721817151521946975147944057562441390161724883072568806497785
# e = 65537

# Use long_to_bytes to convert long to byte string
```

One of thing that immediately pops out is that the primes generated are not big enough, most rsa keys generated today use 512 bits primes.

So what we can do is factorize, but how? There's several fast factorization algorithms out there, one include this [online factorization calculator](https://www.alpertron.com.ar/ECM.HTM).

After running for a few minutes, it spits out the prime factorization,

```
277374 920610 630854 709341 081430 070381 895040 367159 247831 346063 113357 346996 861997 096842 985380 163195 142293 041723 827605 803582 959095 385096 852674 868844 481299 485182 299736 902846 354208 891395 068488 674159 312782 402962 289252 130974 324110 540905 208754 122079 (240 digits) = 698596 659335 109546 655087 × 708633 478546 320284 028527 × 712171 042131 982369 923929 × 775896 948193 047837 060587 × 831556 289895 532903 570177 × 947752 354722 045438 219683 × 997082 735664 824456 059337 × 1 057424 385804 000093 149971 × 1 065963 526540 893590 559577 × 1 144778 007744 688039 498093
```

Another fun thing is it also spits out the euler toitent which we can use.

```
Euler's totient: 277374 920610 630854 709337 876762 735288 861520 145785 756348 680544 006214 182266 297746 719793 060373 981161 425160 348525 569567 062631 851492 462968 215069 409279 551013 961721 009262 754617 684553 993287 757472 034564 148972 521880 994069 518433 487487 039507 563534 090240 (240 digits)
```

We could also calculate the toitent (phi) ourselves by

```
phi = (p1-1)*(p2-1)*(p3-1)...
```

We can now follow rsa, calculate the private key (`d`) and decrypt the flag

```python
from Crypto.Util.number import inverse, long_to_bytes
N = 277374920610630854709341081430070381895040367159247831346063113357346996861997096842985380163195142293041723827605803582959095385096852674868844481299485182299736902846354208891395068488674159312782402962289252130974324110540905208754122079
c = 227688146150768027567515600674768662657437530439630813908158076963410580173896425412413514242614358728441978777344374383817175868212259820714403179898822987968118941373698837582673721817151521946975147944057562441390161724883072568806497785
phi = 277374920610630854709337876762735288861520145785756348680544006214182266297746719793060373981161425160348525569567062631851492462968215069409279551013961721009262754617684553993287757472034564148972521880994069518433487487039507563534090240
e = 65537
d = inverse(e,phi)
print(long_to_bytes(pow(c,d,N)))
# CTF{ECM_factorization_is_magic}
```

`CTF{ECM_factorization_is_magic}`

# Unknown Input
![](https://img.shields.io/badge/category-reversing-blue)

Here is the problem code and given output

```python
#!/usr/bin/env python3
import base64
import os
import random

f = open('flag.txt', 'r')
flag = f.read()
flag = flag.encode('ASCII')
flag = base64.b64encode(flag)
flag = flag.decode('ASCII')

def enc1(text, key):
    key = key*14
    out = ""
    for i,j in zip(text,key):
            out += chr(ord(i)^j)
    return out

def enc2(text) -> str:
    indexes = [10, 5, 26, 2, 0, 24, 9, 27, 16, 11, 14, 12, 8, 25, 4, 15, 21, 22, 20, 18, 13, 1, 23, 6, 17, 7, 3, 19]
    out = ""
    for i in indexes:
        out += text[i]
    return out

def enc3(text) -> str:
    indexes = [11, 13, 27, 16, 8, 21, 20, 26, 4, 12, 24, 19, 23, 17, 15, 7, 10, 0, 25, 18, 14, 5, 1, 9, 22, 3, 2, 6]
    out = ""
    for i in indexes:
        out += text[-i]
    return out

key = os.urandom(2)
for i in enc2(enc3(enc2(enc1(flag, key)))):
    print(hex(ord(i))[2:], end=' ')
```

```
7d 4a 34 9 7e 24 2b 15 48 42 3b 4f 24 1a 1e 42 6e 4 49 7e 4f 15 19 61 62 40 62 7a
```

## Program Analysis

1. Reads flag from hidden file
2. Base64 encodes the flag
3. Runs the function `enc1` with a random 2 byte key from `urandom` on the Base64 encoded flag
4. Runs the function `enc2`
5. Runs the function `enc3`
6. Runs the function `enc2` again
7. Writes to console in hexadecimal

## Reversing the Functions

The program uses 3 different 'encryption' methods during the flag obscuration process. Reverse methods can be written for each of these functions to reverse the obscuration.

### Reversing the `enc1` Function

Examination of the `enc1` function shows that it is a simple XOR cipher

```python
def enc1(text, key):
    key = key*14
    out = ""
    for i,j in zip(text,key):
            out += chr(ord(i)^j)
    return out
```

Due to the fact that encryption and decryption of XOR is exactly the same, we can use the `enc1` function in our reverse function. However, in the program `enc1` is used with a 2 character random key so our reverse function will have to brute force the XOR cipher key. This can be easily done using Python's `itertools.product` method.

```python
import itertools

def revEnc1(text) -> str: #encryption method 1 brute force
    possibilities = []
    for key in itertools.product(range(0,256), repeat=2): #loop through all possible 2 byte keys
        key = bytes(key)
        possibilities.append(enc1(text,key))
    return possibilities
```

This function returns all the results of XOR cipher run with all possible 2 byte keys

### Reversing the `enc2` Function

Examination of the `enc2` function shows that it shuffles the order of characters according to the values of the `indexes` list

```python
def enc2(text) -> str:
    indexes = [10, 5, 26, 2, 0, 24, 9, 27, 16, 11, 14, 12, 8, 25, 4, 15, 21, 22, 20, 18, 13, 1, 23, 6, 17, 7, 3, 19]
    out = ""
    for i in indexes:
        out += text[i]
    return out
```

The function can be reversed by simply creating a function with the same `indexes` list but running a reverse order assignment

```python
def revEnc2(text) -> str: #reverse encryption method 2
    indexes = [10, 5, 26, 2, 0, 24, 9, 27, 16, 11, 14, 12, 8, 25, 4, 15, 21, 22, 20, 18, 13, 1, 23, 6, 17, 7, 3, 19]
    out = [None]*len(text)
    for i in range(len(text)):
        out[indexes[i]] = text[i]
    return "".join(out)
```

### Reversing the `enc3` Function

Examination of the `enc3` function shows that it shuffles the order of characters similar to the `enc2` function. However, this function uses negative indexes which in Python start from the end of the string and not at the beginning.

```python
def enc3(text) -> str:
    indexes = [11, 13, 27, 16, 8, 21, 20, 26, 4, 12, 24, 19, 23, 17, 15, 7, 10, 0, 25, 18, 14, 5, 1, 9, 22, 3, 2, 6]
    out = ""
    for i in indexes:
        out += text[-i]
    return out
```

The reverse function is mostly the same as our `revEnc2` function, just with negative indexes.

```python
def revEnc3(text) -> str: #reverse encryption method 3
    indexes = [11, 13, 27, 16, 8, 21, 20, 26, 4, 12, 24, 19, 23, 17, 15, 7, 10, 0, 25, 18, 14, 5, 1, 9, 22, 3, 2, 6]
    out = [None]*len(text)
    for i in range(len(text)):
        out[-indexes[i]] = text[i]
    return "".join(out)
```

## Reversing the Program

First thing to do is to take the hexadecimal output and turn it into a string. The code to do that looks like this:

```python
ciphertext = input()
ciphertext = ciphertext.split() #convert hexdump to string
for i in range(len(ciphertext)):
    ciphertext[i] = str(chr(int(ciphertext[i], base=16)))
ciphertext = "".join(ciphertext)
```

Next, the `enc2` and `enc3` steps can be reversed by using our `revEnc2` and `revEnc3` functions.

```python
ciphertext = revEnc2(revEnc3(revEnc2(ciphertext))) #reverse remap operations
```

The next thing to do is to brute force the XOR cipher to get all the possibilities

```python
possibilities = revEnc1(ciphertext) #brute force xor cipher
```

Finally, to find our flag we just try to decode all the possibilities with Base64. By catching `UnicodeEncodeError`, `ValueError`, and `binascii.Error` we can skip all improper Base64 strings and use the flag format of `CTF{[A-Za-z0-9_]{6,64}}` to filter for the flag.

```python
for i in possibilities:
    try:
        plaintext = base64.b64decode(i).decode('ASCII') #decode from base64 to ascii
        if plaintext.startswith('CTF{') and plaintext.endswith('}'): #filter for flag
            print(plaintext)
    except binascii.Error: #catch improper base64 encodes
        continue
    except UnicodeEncodeError:
        continue
    except ValueError:
        continue
```

## Final Program Code

```python
#!/usr/bin/env python3
import itertools
import base64
import binascii
import random

def enc1(text, key): # xor cipher
    key = key*14
    out = ""
    for i,j in zip(text,key):
            out += chr(ord(i)^j)
    return out

def revEnc1(text) -> str: #encryption method 1 brute force
    possibilities = []
    for key in itertools.product(range(0,256), repeat=2): #loop through all possible 2 byte keys
        key = bytes(key)
        possibilities.append(enc1(text,key))
    return possibilities

def enc2(text) -> str: #character order remap
    indexes = [10, 5, 26, 2, 0, 24, 9, 27, 16, 11, 14, 12, 8, 25, 4, 15, 21, 22, 20, 18, 13, 1, 23, 6, 17, 7, 3, 19]
    out = ""
    for i in indexes:
        out += text[i]
    return out

def revEnc2(text) -> str: #reverse encryption method 2
    indexes = [10, 5, 26, 2, 0, 24, 9, 27, 16, 11, 14, 12, 8, 25, 4, 15, 21, 22, 20, 18, 13, 1, 23, 6, 17, 7, 3, 19]
    out = [None]*len(text)
    for i in range(len(text)):
        out[indexes[i]] = text[i]
    return "".join(out)

def enc3(text) -> str: #character order remap backwards
    indexes = [11, 13, 27, 16, 8, 21, 20, 26, 4, 12, 24, 19, 23, 17, 15, 7, 10, 0, 25, 18, 14, 5, 1, 9, 22, 3, 2, 6]
    out = ""
    for i in indexes:
        out += text[-i]
    return out

def revEnc3(text) -> str: #reverse encryption method 3
    indexes = [11, 13, 27, 16, 8, 21, 20, 26, 4, 12, 24, 19, 23, 17, 15, 7, 10, 0, 25, 18, 14, 5, 1, 9, 22, 3, 2, 6]
    out = [None]*len(text)
    for i in range(len(text)):
        out[-indexes[i]] = text[i]
    return "".join(out)


ciphertext = input()
ciphertext = ciphertext.split() #convert hexdump to string
for i in range(len(ciphertext)):
    ciphertext[i] = str(chr(int(ciphertext[i], base=16)))
ciphertext = "".join(ciphertext)

ciphertext = revEnc2(revEnc3(revEnc2(ciphertext))) #reverse remap operations
possibilities = revEnc1(ciphertext) #brute force xor cipher

for i in possibilities:
    try:
        plaintext = base64.b64decode(i).decode('ASCII') #decode from base64 to ascii
        if plaintext.startswith('CTF{') and plaintext.endswith('}'): #filter for flag
            print(plaintext)
    except binascii.Error: #catch improper base64 encodes
        continue
    except UnicodeEncodeError:
        continue
    except ValueError:
        continue
```


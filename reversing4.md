# Bad Bunny
![](https://img.shields.io/badge/category-reversing-blue)

I recently got together with others to write some challenges for a CTF competition. I ended up writing two problems: a reverse engineering one and a binary exploitation one. Today I'll be writing about the construction and solution of the reverse engineering one.

This was the first time I have ever written my own problem for a CTF so it was far from perfect but proved to be a nice challenge for competitors. Also, giving credit where credit is due, much of this problem took inspiration from a similar one in the Canadian Communications Security Establishment's Geek Seek CTF.

### The Start

Competitors were first given [this file](bunny.pyc) to begin. Guessing from the extension or base64 encoded contents within, this was a clearly a byte-compiled Python file, but which version? Well, initially I had used Python 3.7 for this challenge because it was widely used as the default across Linux distributions and because of the numerous decompilers available. However, in a last minute change, I switched to Python 3.9 which was neither widely installed nor had decompilers available. This switch made the initial part of the challenge, understanding how to run the program harder, but not too difficult with some trial and error. Proceeding with the correct version gave the following output:[^1]

[^1]: Decoding the base64 encoded strings in the file could also reveal the same prompt.

```
λ python bunny.pyc

   /\   /\     The rabbit has stolen your flag and hid it in its hat! Can you
   \ \_/ /     give it the right code to get your flag back?
    (`Y')
   ()~*~()
  ▃▃▃▃▃▃▃▃▃
   ▉▉▉▉▉▉▉
   ▉▉▉▉▉▉▉
   ▉▉▉▉▉▉▉     Rabbit unlock code:
```

Attempting to enter an incorrect code gives the following output:

```
  ▃▃▃▃▃▃▃▃▃
   ▉▉▉▉▉▉▉
   ▉▉▉▉▉▉▉   Wrong code. The rabbit dips its head back into the hat.
   ▉▉▉▉▉▉▉
```

Expected.

### So what next?

Understanding what the program actually does! Given that no decompiler for Python 3.9 existed, a few approaches could be tried. Some competitors read the Python bytecode to reconstruct the code, while others cleverly used the `strace` utility to figure out everything necessary that went on. Analysis of the binary itself could also reveal a function named `derive_key` with some nearby strings and variables like `password`, as well as a `real_derive_key` function... in a section of the file after an `ELF` header? What's up with that? For that, let's look at the original source code of the program (with an `exec` call expanded):

```py
#!/usr/bin/env python3
import base64, ctypes, sys, os
from cryptography.fernet import Fernet

CIPHERTEXT = b"gAAAAABf8k6VO2v1na8YaHGUkUfHF-iwtVIfyzt5qyhklQUCT3WCLvZuiVy-dkWoXvJf6ssP9PJE0HIYluIitJf_mQQUtu9vXMAaT_JSc9i5aRd5LQkmQ3GDMcLHSgjvfWuAYpM5dJled2l0aCBvcGVuKHN5cy5hcmd2WzBdLCAicmIiKSBhcyBmOgogICAgZi5zZWVrKDIwOTEpCiAgICB3aXRoIG9wZW4oIi90bXAvbGlicmVhbC5zbyIsICJ3YiIpIGFzIGZmOgogICAgICAgIGZmLndyaXRlKGYucmVhZCgpKQogICAgbGliID0gY3R5cGVzLkNETEwoIi90bXAvbGlicmVhbC5zbyIpCiAgICBsaWIubGliLnJlc3R5cGUgPSBjdHlwZXMuY19jaGFyX3AKICAgIGV4ZWMobGliLmxpYigpLmRlY29kZSgpKQ=="
BANNER = base64.b64decode(b"CiAgIC9cICAgL1wgICAgIFRoZSByYWJiaXQgaGFzIHN0b2xlbiB5b3VyIGZsYWcgYW5kIGhpZCBpdCBpbiBpdHMgaGF0ISBDYW4geW91CiAgIFwgXF8vIC8gICAgIGdpdmUgaXQgdGhlIHJpZ2h0IGNvZGUgdG8gZ2V0IHlvdXIgZmxhZyBiYWNrPwogICAgKGBZJykKICAgKCl+Kn4oKQogIOKWg+KWg+KWg+KWg+KWg+KWg+KWg+KWg+KWgwogICDilonilonilonilonilonilonilokKICAg4paJ4paJ4paJ4paJ4paJ4paJ4paJCiAgIOKWieKWieKWieKWieKWieKWieKWiSAgICAgUmFiYml0IHVubG9jayBjb2RlOiA=").decode()
GOOD = base64.b64decode(b"CiAgIC9cICAgL1wKICAgXCBcXy8gLyAgICAgVGhlIHJhYmJpdCBqdW1wcyBvdXQgb2YgdGhlIGhhZCwgY29uZ3JhdHVsYXRpbmcgeW91IGZvciB0aGUgY29ycmVjdAogICAgKCd1JykgICAgICBjb2RlLgogICAoKX4qfigpCiAgIChfKS0oXykgICAgIEhlcmUncyB5b3VyIGZsYWc6IA==").decode()
BAD = base64.b64decode(b"CiAg4paD4paD4paD4paD4paD4paD4paD4paD4paDCiAgIOKWieKWieKWieKWieKWieKWieKWiQogICDilonilonilonilonilonilonilokgICBXcm9uZyBjb2RlLiBUaGUgcmFiYml0IGRpcHMgaXRzIGhlYWQgYmFjayBpbnRvIHRoZSBoYXQuCiAgIOKWieKWieKWieKWieKWieKWieKWiQ==").decode()

def derive_key(password):
    if password == "mvrynIDAX4tcQbp9wyTNPbBLldKiG0IJ":
        return b"znGy45To72bqSDW96Zoj5I5tZxeDKrFgORpngtKzHhRz"
    return ""

# Actually: exec(base64.b64decode(CIPHERTEXT[140:]))
with open(sys.argv[0], "rb") as f:
    f.seek(2091)
    with open("/tmp/libreal.so", "wb") as ff:
        ff.write(f.read())
    lib = ctypes.CDLL("/tmp/libreal.so")
    lib.lib.restype = ctypes.c_char_p
    exec(lib.lib().decode())

def decrypt(password):
    key = derive_key(password.encode())
    if key:
        return GOOD + Fernet(key).decrypt(CIPHERTEXT[:140]).decode()
    return BAD

print(decrypt(input(BANNER)))
```

Aha! So it seems like this program takes an input and passes it to `derive_key`, then attempts to perform a Fernet decryption. Trying the password visible here doesn't actually work though, and looking a little further to the expanded `exec` call will soon reveal why. It's clear that a CDLL is loaded from an offset in the original file itself. It can be found by simply going to that offset or reading some `strace` output.

*Creator note: this file was just made by concatenating the Python byte-compiled file and this Shared Object file.*

### Conclusion

Ultimately, having this new binary, [`libreal.so`](libreal.so) should be enough to solve the challenge, even if one were to completely ignore the Python aspect. Disassembling or decompiling it with a variety of tools (such as Ghidra or Radare2) reveals the following code:

```c
#include <string.h>
#include <stdlib.h>

// Correct pass: KAHQ7d1lb
char* real_derive_key(char* pass) {
	if (pass[0] + pass[7] == 183) {
		if (pass[3] + pass[5] == 181) {
			if (pass[2] * pass[3] == 5832) {
				if (pass[1] - pass[8] == -33) {
					if (pass[4] + pass[2] == 127) {
						if (pass[6] * pass[7] == 5292) {
							if (pass[4] + pass[1] == 120) {
								if (pass[6] - pass[5] == -51) {
									if (pass[4]-48 + pass[6]-48 == 8) {
										char *dest = malloc(44*sizeof(char));
										strcpy(dest, pass);
										strcat(dest, "IM4bcDAvcxdnkLmtM-388TiRI1XAgn7C6I=");
										return dest;
									}
								}
							}
						}
					}
				}
			}
		}
	}
	return "";
}

char* lib() {
	return "derive_key = lib.real_derive_key\n"
			 "derive_key.argtypes = [ctypes.c_char_p]\n"
			 "derive_key.restype = ctypes.c_char_p\n"
			 "os.remove(\"/tmp/libreal.so\")";

}
```

That `lib()` function at the bottom is what actually reveals why the original password doesn't work. The `derive_key` function gets replaced with this `real_derive_key` function in C. Figuring out the correct input here can be done with some brute forcing, math, or a SAT solver like [Z3](https://github.com/Z3Prover/z3), revealing `KAHQ7d1lb`[^2] as the correct input and the flag:[^3]

[^2]: I haven't actually checked if other correct solutions for the real function exist, but those could be bruteforced anyways.
[^3]: The reference to [Cython](https://cython.org/) in the flag had to do with previous ideas I had for this challenge. I changed it locally but forgot to do so for the version sent to competitors, oh well.

```
   /\   /\
   \ \_/ /     The rabbit jumps out of the had, congratulating you for the correct
    ('u')      code.
   ()~*~()
   (_)-(_)     Here's your flag: CTF{D0n7_w3_4ll_l0v3_50M3_Cy7h0n_fUn}
```

### Reflection

Ultimately, I think this challenge was *decent* for my first one ever. It had many solutions allowing for some great creativity while still proving difficult enough that only a few people succeeded in completing it.

The switch from Python 3.7 to 3.9 is something I'm not completely certain about though. The very first steps of the challenge weren't originally meant to be the most difficult, but I ran out of time and knowledge to figure out a more creative way to hide the overwritten function. Letting contestants know that one had to use Python 3.9 to run the challenge was probably something I should have done because it would have made the experience smoother for everyone involved.

However, despite the issues, I certainly learned a lot by making this challenge and hope to make some better problems in the future!

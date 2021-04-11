# CTFplatform
![](https://img.shields.io/badge/category-reversing-blue)

## Description
**jdabtieu**’s boss asked him to write some pseudocode for some random program. Since **jdabtieu**’s primary programming language is not C, nor Java, or C++, or even Python, but Assembly, he decided to write pseudo-assembly-code. After all, pseudocode is just code but more english, right?

When he presented this pseudo-assembly-code to his boss **Maillew**, **Maillew** flipped a lid and asked how he was supposed to read this, since **Maillew** only understands C++ code and this was not traditional pseudocode either. So, **Maillew** has tasked you to figure out what on earth this pseudo-assembly-code does.

Unimpressed by the abysmal syntax, you open up your favourite text editor (vim, of course) and begin typing away… in assembly.

## Hints
Certain instructions make it almost impossible to directly translate into a higher level language like Python.

## Solution
While it is possible to convert this psuedo-assembly-code into actual assembly code, it would take a quite large understanding of assembly to know how to fix it, and at that point, you might as well just read it by hand since the file is small enough.

There are a few things pieces of background information you must first know about assembly:
1. In 64-bit assembly, functions parameters are passed in the registers rdi, rsi, rcx, rdi, r8, r9, in that order. Compared to C, this would be `return_type function_name(param1 --> rdi, param2 --> rsi, etc.)`
2. Most registers have 32-bit, 16-bit, and 8-bit versions. For example, rax is 64-bits, eax is the lower 32-bits of rax, ax is the lower 16-bits of rax, and al is the lower 8-bytes of rax. Generally, r_x is a 64-bit register, e_x is 32-bit, _x is 16-bit, and _l is 8-bit.

We can now start reading through the assembly.

```
        load    ?_001
        call    puts  ; puts(str) prints str to stdout
```
This appears to load whatever the value of ?_001 is into rdi, and then prints it. We can see at the bottom of the file, that `?_001` contains the bytes `0x43`, `0x54`, `0x46`, and `0x00`. Converting to ASCII, we get `CTF` (plus a null byte).

```
        move    16-bit [rbp-2H], 123
        load    rdi, [rbp-2H]
        call    puts
```
This appears to move the value 123 into some 16-bit location called [rbp-2H], and then loads it to be printed. Once again, referring to an ASCII table, we can see that it corresponds to a `{`.

```
        move    8-bit [rbp-2H], 99
        and     8-bit [rbp-2H], 89
        load    rdi, [rbp-2H]
        call    puts
```
This time, it looks like the variable is set to the bitwise AND of 99 and 89. (In most programming languages, the syntax would be `99 & 89`). Thirs results in 65, which is `A` on the ASCII table.

```
        move    32-bit [rbp-6H], 21
        move    32-bit [rbp-0AH], 0
        goto    ?_003
?_002:  add     32-bit [rbp-6H], 2
        add     32-bit [rbp-0AH], 1
?_003:  cmp     32-bit [rbp-0AH], 16
        jl      ?_002
```
It looks like 21 is stored in a variable, and 0 in another. We'll call these variables `a` and `i` for now. After setting these variables, it jumps to `?_003`, which compares `i` to 16 and jumps to `?_002` if i < 16. This is indeed a loop, and will run 16 times. It adds 2 to `a` 16 times, so `a`'s final value would be 21+2*16=53 and `i`'s final value would be 16.

In C, this can be represented as such:
```c
int a = 21;
for (int i = 0; i < 16; i++) {
    a += 2;
}
```

Moving on...
```
        move    eax, 32-bit [rbp-6H]
        move    8-bit [rbp-2H], al
        load    rdi, [rbp-2H]
        call    puts
```
[rbp-6H] - that's a. The value of `a` is moved to eax, and then the lower 8 bits is moved to [rbp-2H]. Then we load that and print it. Effectively, we are printing `a % 256`, since the lowest 8 bits can store the numbers 0-255. Referring to the ASCII talbe again, 53 corresponds to the character `5`. Our flag so far is `CTF{A5`.

```
        move    8-bit [rbp-2H], 4
        cmp     8-bit [rbp-2H], 20
        jg      ?_002
        goto    ?_004
?_005:  move    8-bit [rbp-2H], 77
        load    rdi, [rbp-2H]
        goto    ?_007
?_006:  move    8-bit [rbp-2H], 56
        load    rdi, [rbp-2H]
        goto    ?_007
?_004:  cmp     rdi, 44
        jge     ?_005
        goto    ?_006
?_007:  call    puts
```
This compares 4 to 20, and jumps to `?_002` if it's true. Obviously, we know that 4 is not greater than 20, so we do not jump to `?_002`. Instead, we jump to `?_004`. This compares rdi to 44. The last time we set rdi, we set it to the address of [rbp-2H] using load (the memory address would most certainly be larger than 44). However, the value of rdi is allowed to change after a function call (we called puts), so it may not necessarily be still be that address. Considering that we did not get the source code for puts though, we'll just have to assume it does keep the old value. We jump to `?_005`. Here, we load and print 77, which is `M` in ASCII.

```
        move    8-bit [rbp-2H], 117
        xor     8-bit [rbp-2H], 42
        load    rdi, [rbp-2H]
        call    puts
```
Next, we XOR 117 and 42, and print it. Using a calculator, we get 95, correspoding to `_`.
```
        move    32-bit [rbp-4EH], 1
        move    32-bit [rbp-4AH], 51
        move    32-bit [rbp-46H], 6
        move    32-bit [rbp-42H], 1
        move    32-bit [rbp-3EH], 26
        move    32-bit [rbp-3AH], 23
        move    32-bit [rbp-36H], 26
        move    32-bit [rbp-32H], 1
        move    32-bit [rbp-2EH], 61
        move    32-bit [rbp-2AH], 1
        move    32-bit [rbp-26H], 2
        move    32-bit [rbp-22H], 71
        move    32-bit [rbp-1EH], 6
        move    32-bit [rbp-1AH], 26
        move    32-bit [rbp-16H], 26
        move    32-bit [rbp-12H], 2
        move    32-bit [rbp-0EH], 0
        move    32-bit [rbp-0AH], 0
```
This constructs an array of integers.
```
?_008:  cmp     32-bit [rbp-0AH], 16
        jge     ?_009
        move    eax, 32-bit [rbp-0EH]
        convert eax to rax  ; (damn it i forgot what the instruction is called)
        move    rdx, rax
        move    eax, 32-bit [rbp-0AH]
        convert eax to rax  ; (damn it i forgot what the instruction is called)
        cmp     32-bit [rbp-4EH+4*rax], edx
        jle     ?_010
        move    edx, 32-bit [rbp-4EH+4*rax]
        move    32-bit [rbp-0EH], edx
?_010:  inc     32-bit [rbp-0AH]
        goto    ?_008
```
Here it looks like we have some sort of loop again. This loop is run 16 times, and jumps to ?_009 if [rbp-0AH] >= 16. The next 3 lines moves [rbp-0EH] into rdx and [rbp-0AH] (which I'll refer to as the loop index counter) into rax. `32-bit [rbp-4EH+4*rax]` accesses `arr[i]` because `rbp-4EH` is the start of the array, rax=i, and each item in the array is 4 bytes. It looks like it checks if the item in the array is greater than edx, and sets edx equal to that item if it is. Effectively, this loop finds the maximum element in the array and stores it in rbp-0EH.
```
?_009:  dec     32-bit [rbp-0EH]
        move    eax, 32-bit [rbp-0EH]
        move    8-bit [rbp-2H], al
        load    rdi, [rbp-2H]
        call    puts
```
Next,  it subtracts 1 from the max of the array (which would be 71) to get 70, and prints it. In ASCII, this is `F`.
```
        move    8-bit [rbp-4H], 125
        move    8-bit [rbp-3H], 78
        move    8-bit [rbp-2H], 85
        move    al, 8-bit [rbp-2H]
        move    dl, 8-bit [rbp-4H]
        move    8-bit [rbp-4H], al
        move    8-bit [rbp-2H], dl
```
This creates a string with ASCII values 125, 78, 85 (`}NU`) and then swaps the first and third characters, to get `UN}`.
```
        load    rdi, [rbp-4H]
        call    puts
        move    eax, 0
        end lmao
```
We then load `UN}` and print it. The final flag is `CTF{A5M_FUN}`

Flag: `CTF{A5M_FUN}`

# Reading Comprehension
![](https://img.shields.io/badge/category-general-blue)

## Description

You are to find the very secretive message hidden in this text. The flag will contain 6 lowercase words concatenated. Good luck.
```
In this ctf,
the flag will be
six numbers long and in word format.
The words will be concatenated and will be
mixed into a string that will be the flag.
The flag is thirty-four.
```
## Solution

Using reading comprehension skills, the trivial flag is `ctf{threefour}`. However, upon further inspection using more reading comprehension skills, one can determine that the flag is actually 6 words long, each of which is a number.

Notice that the text is actually given in 6 different lines. Each line has a certain number of words. Thus, one can deduce that the flag is `ctf{threefourseveneightninefour}`

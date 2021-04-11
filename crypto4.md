# =~('(?{'.('/-)@_{'^'__@.+[').'"'.('@*-:[-}'^'-__^>__').',$/})')
![](https://img.shields.io/badge/category-crypto-blue)

## Description

Yesterday, you found a unique-looking USB in your driveway. Going against security best practices and your instincts, you decided to plug it into your desktop computer. Luckily, it didn’t immediately shock and permanently damage your desktop computer. You then analysed the disk, hoping to find something of interest.

The data on the disk appeared statistically random—usually a good sign that there was something hidden. It could be a Bitcoin wallet, or thousands of health records. You made an image of the drive and continued to analyse it…

[crypto4.zip](https://drive.google.com/drive/folders/1eKV2WG1MR48uCN-uw6tAp6vYV6iZHcde?usp=sharing)

## Hint

VeraCrypt was developed from TrueCrypt. How different are they, really?

## Solution

The hints provided practically scream at us that VeraCrypt was used and we should John Jumbo to crack it. It turns out that there is no script to crack VeraCrypt, but research indicates that VeraCrypt was developed from TrueCrypt, and John Jumbo has a script to crack TrueCrypt. We try that. We extract the hashes with `truecrypt2john.py` and crack them with the wordlist provided in the Discord.

## Flag

`CTF{and_W3_Ar3_M3r3Ly_pLaY3r2}`

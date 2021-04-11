# The Human Factor
![](https://img.shields.io/badge/category-crypto-blue)

## Description

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512
```
You may not be familiar with PGP. According to Wikipedia:

"Pretty Good Privacy (PGP) is an encryption program that provides cryptographic privacy and authentication for data communication. PGP is used for signing, encrypting, and decrypting texts, e-mails, files, directories, and whole disk partitions and to increase the security of e-mail communications."

Even though there have been flaws found in implementations, the underlying cryptography has been working perfectly since PGP was invented. Your job today is to break PGP.
```
-----BEGIN PGP SIGNATURE-----

iQJNBAEBCgA3FiEESMez/LsGDJV+XkaPOBmPu5crdloFAmAHCsAZHGNoZW5hbnRo
b255MzY1QGdtYWlsLmNvbQAKCRA4GY+7lyt2Wpm2D/4g6bVbLGqiPu9mYmHYvSHI
FXfOHgn7M1Mba9HORJ0rD/4qrFnfVWa/ObK+oeJkNCFO0jhLJcAjL8VpnbTtuJib
If9V6Jxnv5ng5jf6LDH/XtXd/i4SW75vQNq4Oijf2qqLzlOZ7hvnk12BsHEOLio+
6Z0jevkiH17KgcZ4+Dt2twaIu6upt/R1K0+9mlnL6mIpWA2+WcLmUyMNj36al64w
KDcwQv32TxvvZmDiNx0T4FP7Ueux06zoYmpheTaZ3BIXxrxTEGr/LHifNM/Nn/nS
IpcOqZaxBg2H3QSYS/GU5DxO60ASaWj5dSVCdp8SorVvTFiiwYBAwkNBeq3Fv7Jy
IOkOge1agl/fryTb9UxT01htILPFL24GM063zoZ48KUtin4A1M72+ZhdepMw37fK
l3iDIfIgDeooYn2hygETXswfJadwEo5Yx6v/O34qV2C1+WM2wOFM16L1xnZkXzQw
u+GsvAyl/18Gonw2O8cgoCXF01HwhX3kDMAiOjSurM3l+jzyEIBd0E/Z8Og33ilq
+QviRi2GyCKg23wSzXGgWQbFO+W9bKWxBgv1nfFLD4xti91G1KIX2oY+mvt/BfHe
BMhgtMJpmTqEKKo9Dx2R8VtIF+df4UEC0JYH4gB7prZULByqCuKCtExSpfFVlbku
DIuOW8kT7yh+QZm+Cw+/IQ==
=dPRJ
-----END PGP SIGNATURE-----
```

[crypto2.zip](https://drive.google.com/drive/folders/1eKV2WG1MR48uCN-uw6tAp6vYV6iZHcde?usp=sharing)

## Hint

A cryptographic system is only as strong as its weakest link.

## Solution

After some research, we find out .asc is the standard file format for PGP-encrypted files. We can use the following command to check the algorithm.

```
$ gpg -vv --show-session-key --list-packets message1.txt.asc 
gpg: armor: BEGIN PGP MESSAGE
# off=0 ctb=8c tag=3 hlen=2 plen=13
:symkey enc packet: version 4, cipher 9, s2k 3, hash 2
	salt 3C35BF07D859A80F, count 65011712 (255)
gpg: AES256 encrypted data
$ gpg -vv --show-session-key --list-packets message2.txt.asc 
gpg: armor: BEGIN PGP MESSAGE
# off=0 ctb=84 tag=1 hlen=2 plen=140
:pubkey enc packet: version 3, algo 1, keyid 1C35AB5111279E04
	data: [1023 bits]
gpg: public key is 1C35AB5111279E04
gpg: using subkey 1C35AB5111279E04 instead of primary key 86149011C0541D05
$ gpg -vv --show-session-key --list-packets message3.txt.asc
gpg: no valid OpenPGP data found.
gpg: processing message failed: Unknown system error
```
We note `message2.txt.asc` is encrypted to the given public/private key pair. We can then check how strong that key is.
```
$ gpg public.pgp
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
pub   rsa1024 2021-01-04 [SC]
      27A9E01241F03C6AA398E18F86149011C0541D05
uid           MGCTF <ctf.mgci@gmail.com>
sub   rsa1024 2021-01-04 [E]
```
Another quick Google search tells us that neither AES-256 or RSA-1024 have been broken yet. Hence, the only possible solution is to try and guess the passphrase used with a brute-force attack. We can use John Jumbo to crack both message1 and message2, since the third file does not appear to be valid. We use a simple wordlist (a list of the top 10000 passwords will do). The rest is trivial, and left as an exercise to the reader. 

## Flag

`CTF{0123456789abcdef}`

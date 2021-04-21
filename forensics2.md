# Rabbit Hole
![](https://img.shields.io/badge/category-forensics-blue)

## Description
Sometimes you just have to follow the trail, even if it has some twists and turns...

## Solution
The file we start off with is `dontopenwithimageviewer.jpg`. We should follow the advice on the name and open it with Notepad instead, we can find the link `https://tinyurl.com/itisnotthemostobviouschoice` at the bottom. 

Following the link (which may not be available at this point) gives another image called `lol_gottem.png`, which is a barcode. Decoding the barcode is a red herring :blobcreep:, but using a steganography tool on the image would reveal a link to a .wav file (`http://138.197.69.9/dl/forensics2/shifted_flag.wav`). 

This file contains a Morse code message, which is `KNSFQIJXYNSFYNTS`. Bruteforceing Caesar Cipher would give `FINALDESTINATION`, which is our flag.

`CTF{FINALDESTINATION}`

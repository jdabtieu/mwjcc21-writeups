# Remarks
![](https://img.shields.io/badge/category-forensics-blue)

## Description
A friend did this contest that got rave reviews; he sent me one of them and muttered something about a double meaning, but I can't quite figure out what it is...

## Solution
The screenshot implies that the flag may be hidden as a comment in the image's metadata. Run `exiftool` to get the following metadata entries:
```
ExifTool Version Number         : 12.16
File Name                       : screenshot.png
Directory                       : .
File Size                       : 13 KiB
File Modification Date/Time     : 2021:04:04 13:01:54-04:00
File Access Date/Time           : 2021:04:04 13:01:54-04:00
File Creation Date/Time         : 2021:04:04 13:01:52-04:00
File Permissions                : rw-rw-rw-
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 403
Image Height                    : 105
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
SRGB Rendering                  : Perceptual
Gamma                           : 2.2
Pixels Per Unit X               : 6614
Pixels Per Unit Y               : 6614
Pixel Units                     : meters
Comment                         : ezU2NDYwMzM2X3NlZ2FNaV9ydTB5X3RuZW1tMGN9RlRD
Image Size                      : 403x105
Megapixels                      : 0.042
```
Indeed something suspicious is in the comment. Looks like base64: 
```
> echo "ezU2NDYwMzM2X3NlZ2FNaV9ydTB5X3RuZW1tMGN9RlRD" | base64 -d
{56460336_segaMi_ru0y_tnemm0c}FTC
```
Reverse the string to get the flag: `CTF{c0mment_y0ur_iMages_63306465}`

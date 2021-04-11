# Forgetful
![](https://img.shields.io/badge/category-forensics-blue)

## Description
Some while ago, I installed this photo viewer script that regularly pops up a window with my favourite photo on it! However, one day I shuffled my files around and it stopped working. I can't remember what was on the photo anymore... could you tell me?

## Hint
My wastebasket is overflowing...

## Solution
The file attached is a memory dump, so `volatility` could be a good tool to solve this problem.

In order to perform analysis, `volatility` requires the image's "profile", so run `volatility -f mem.vmem imageinfo` (`mem.vmem` being the file name) to determine it:
```
Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
```
Any of the suggested profiles should do.

Next, this "photo viewer script" referenced suggests a PowerShell or Batch script. Try looking for files that have their respective extensions:
```
> volatility -f mem.vmem --profile Win7SP0x86 filescan | grep -i .ps1
Volatility Foundation Volatility Framework 2.6
0x000000003e4387a0      6      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\Help.format.ps1xml
0x000000003e439620      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\PowerShellCore.format.ps1xml
0x000000003e6e9718      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\PowerShellTrace.format.ps1xml
0x000000003e7e2330      6      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\types.ps1xml
0x000000003e7e24f8      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\typesv3.ps1xml
0x000000003e7e2768      2      0 R--r-- \Device\HarddiskVolume1\Users\monkey\Documents\picture_frame\picture_frame.ps1
0x000000003e7f2230      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\Certificate.format.ps1xml
0x000000003e7f41d0      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\HelpV3.format.ps1xml
0x000000003e9718c0      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\Registry.format.ps1xml
0x000000003e9f5740      7      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\DotNetTypes.format.ps1xml
0x000000003e9f5cc0      8      0 R--r-- \Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\FileSystem.format.ps1xml
```

`picture_frame.ps1` seems interesting; dump the file to see its contents:

```
> volatility -f mem.vmem --profile Win7SP0x86 dumpfiles -Q 0x000000003e7e2768 --dump-dir temp/
> cat temp/file.None.0x85be26a8.dat
Add-Type -A System.Windows.Forms
[System.Windows.Forms.Application]::EnableVisualStyles()

# %rags%
$src="$env:rags\rrr.png"

$W=New-Object System.Windows.Forms.Form
$W.ClientSize=New-Object System.Drawing.Point(400,250)
$W.text="preview"
$W.Icon=[system.drawing.icon]::ExtractAssociatedIcon("$PSScriptRoot\flag.ico")

if (test-path $src) {
 $W.BackgroundImage=[system.drawing.image]::FromFile($img_src)
}
else {
 $L=New-Object System.Windows.Forms.Label
 $L.text="ERR: can't find the image?"
 $L.AutoSize=$true
 $L.location=New-Object System.Drawing.Point(14,20)
 $L.Font='Microsoft Sans Serif,15'
 $W.controls.AddRange(@($L))
}

[void]$W.ShowDialog()
```
(Yes, `$img_src` should be `$src` but that does not affect this problem). 
Indeed this is a script that pops up a photo and `$env:rags\rrr.png` seems to be the path to the "favourite photo" referenced in the problem. However, `grep`ping the `filescan` results for `rrr.png` doesn't seem to yield the photo...
```
> volatility -f mem.vmem --profile Win7SP0x86 filescan | grep -i rrr.png
Volatility Foundation Volatility Framework 2.6
0x000000003e6d0ec0      8      0 -W-rwd \Device\HarddiskVolume1\$Recycle.Bin\S-1-5-21-2547459319-2975066503-1130160202-1014\$RN0BRRR.pngpng
0x000000003fafc578      8      0 -W---- \Device\HarddiskVolume1\$Recycle.Bin\S-1-5-21-2547459319-2975066503-1130160202-1014\$IN0BRRR.png
```

Perhaps the "shuffling files" in the description meant that the file was deleted. The hint implies this as well. 
*A non-hint route to further suggest this is that in the script, `$env:rags` implies that `rags` is an environment variable; using volatility's `envars` command the value of `rags` is revealed to be `C:\Users\monkey\Pictures\stock`. `grep`ping the `filescan` for that path does not yield any results.*

Checking the Recycle Bin contents yields many files - more specifically, over a thousand. One *possible* solution is checking each file for possible `png` contents, but such gruelling labour is not necessary.

Each recycled item in the modern Windows Recycle Bin is represented by two files: one prefixed with `$R`, which contains the original item's contents, and the other with `$I`, which contains its metadata, such as the original file name and path location. The remaining part of both file names consists of random characters, with the same suffix for each pair of files. The path to each user's Recycle Bin is `C:\$Recycle.Bin\SID\`.
For example, if the files in the bin were  `$IN0BRRR.png $RN0BRRR.png.png $I3NNT0X.png $R3NNT0X.png`, `$IN0BRRR.png` and `$RN0BRRR.png.png` represent one recycled file and `$I3NNT0X.png` and `$R3NNT0X.png` represent another recycled file. 

With that in mind, the remainder of this solution is as follows:
1. Dump all of the files inside the recycle bin (so their file path contains `$Recycle.Bin`) via the `dumpfiles` command. 
2. Write a script to perform the following:
	1. Check all of the files whose original name was prefixed `$I` for `\monkey\Pictures\stock\rrr.png` in its contents. 
	2. Find the corresponding `$R` file and view the `png`'s contents. 

Flag: `ctf{r3cycLers_missing}`

# CTFplatform
![](https://img.shields.io/badge/category-pwn-blue)

## Description
A weird clone of MGCI’s CTF Club website has popped up at [http://159.203.32.136:30003](http://159.203.32.136:30003) from an unknown source. Who created it? Is it just a phishing attempt? Is it a CS student trying to play around with the source code? **jdabtieu**, the president of MGCI's CTF Club, has tasked you with finding out what this clone is supposed to do, citing being too busy crying about AMC & CCC to do it himself.

Note: You will have to sign up for an account at that website. You should create a throwaway account using a random username and email.

## Hints
Have you clicked on *all* the links yet? Including the source code link? Potentially the version might be important too? Just a thought…

What's the difference between `rwsrwsr-x` and `rwxrwxr-x`?

## Solution
The first hint tells us to click on the source code link, and pay attention to the version number. Clicking on the GitHub link, we notice that there are two versions released - the latest one is v1.4.0, not v1.3.1. Comparing the difference between the two versions, it appears that a debug comment was removed, something with `ajax` was moved around in the file, a link to `/admin/system` was added, and some HTML was changed. It also mentions some security *issue*. Clicking on the issues tab, we notice one closed issue [here](https://github.com/jdabtieu/CTFplatform/issues/1). Following the steps in the issue, we can give ourselves an admin account.

Once logged in, looking around doesn't result in much. The database looks basically empty, with no problems, contests, announcements, and just two users. However, the `/admin/system` page does exist - the button was just missing in this version.

Typing in `ls -la` as the command, we notice that the flag.txt file exists, but is owned by root and not readable by us. Running `cat flag.txt` gives us the help message for CTShell, that we can only use a few limited commands. Trying to bypass the filter using `ls -la; echo "hi"`, `ls -la && echo "hi"`, and `bad || echo "hi"` don't work.

As expected, `file flag.txt` tells us that it contains ASCII text. But wait - if the file is only readable by root, how does it know that this file contains text? Turns out, if you run `ls -l /usr/bin/file`, the permissions show that this is a SUID binary - that anyone running the command runs it as root, without a password. This is hinted at by the second hint.

To figure out how to leak the flag, we can refer to [GTFObins](https://gtfobins.github.io/), a well-known resource listing programs and their capabilities, including SUID.

Looking under the SUID heading for the `file` command, it lists `file -f file_to_read` as the command to use.

`file -f flag.txt`

`CTF{always_do_security_updates}: cannot open 'CTF{always_do_security_updates}' (No such file or directory)`

Flag: `CTF{always_do_security_updates}`

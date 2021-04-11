# Eyes Wide Open
![](https://img.shields.io/badge/category-general-blue)

## Description
Orson sighed. A few months ago, he had lost his password to his passwords! If he couldn't open his password-protected ZIP in the next month, he would lose his job (and would never be able to buy Holy Pandas >.<). Possibly even more tragic, he would never be able to post cat pictures on Instagram again.

Footsteps. He looked up to see a fellow coworker, also named Orson Kim, walk by. He sighed again; he was always the second. He loved his job at the quarry mining OurCoin, if only he could find that password…

## Hint
When in doubt, https://lmddgtfy.net/?q=osint. If still in doubt, maybe Sherlock can show you a thing or two

## Solution
Given the knowledge this question is intended to be OSINT, we can search the description for any clues. We notice Orson has an Instagram account mentioned, and that he's "always the second". Using this, we find his account to be [orsonkim2](https://instagram.com/orsonkim2).

On his account, we discover a few posts, some highlights, and a reel. Playing the latter, we find in the last few seconds the following message:

> I'M AN AFFILIATE NOW! USE MY CODE:

> `v=kYKsX_HB22Y`

> FOR 10% OFF AT KBDFANS!

with the description as:

> KAHDAOJSJJDAFFILIATE took me exactly 35 minutes but here I am!

You may have previously found [orsonkim](https://instagram.com/orsonkim), which contains a slight hint for the next step: he only follows those with a username connected to "rock" or "coin", except [Susan Wojcicki](https://www.instagram.com/susanwojcicki/) - the CEO of YouTube. Given the code from the reel, we realize it matches the query for a YouTube video. Prepending "youtube.com/watch?", we reach [this video](https://www.youtube.com/watch?v=kYKsX_HB22Y).

Although there's nothing suspicious on the particular video, we take a look at the other videos on the channel. Revisiting the description from the reel, we search for a video longer than 35 minutes, which turns out to be [this video](https://www.youtube.com/watch?v=6eDuPzX0woI). Checking the subtitles at 35:00, we find the code:

> g2k-jL93H_;aacTfpW@eWo

Using this to unlock the zip, we're greeted with a passwords file, a cat gif, and this:
![](https://cdn.discordapp.com/attachments/687649403036893243/825124275714392074/your_eyes_were_opened.png)

Flag: `CTF{L1kE_c0mm3nt_5uBScr18e_j7r3L25M9}`

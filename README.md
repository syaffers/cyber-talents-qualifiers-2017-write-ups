# CyberTalent CTF 2017 Write-ups
I'm using Ubuntu 16.04 for most of these terminal commands. I assume that
it is installed on the system so I don't have to keep putting in
`sudo apt-get install <x>` everywhere.

## This is Sparta (Easy, 50 pts, Web Security)
TODO

## Search in Trash (Easy, 50 pts, Digital Forensics)
We are given some file, no extension. So let's `file` it:

    $ file search-trash
    search-trash: Windows Recycle Bin INFO2 file (Win2k - WinXP)

Okay, so it's an INFO2 recycle bin file, let's see if the flag is directly
visible in the binary through strings:

    $ strings search-trash
    ...
    :\ext.txt
    :\fa.txt
    :\fi.txt
    :\FLag{Fat_32_DF_2}.txt
    :\fr.txt
    :\fur.txt
    :\fy.txt
    ...

Yup, it's there, Ctrl-C, Ctrl-V and done.

## Pure Luck (Basic, 25 pts, Malware Reverse Engineering)
Again, unidentified file `pure-luck.out`. Upon `file`-ing it, it's an
executable, but 32-bit, which can't be run on my 64-bit linux. No worries.
Let's decompile it using IDA Pro. On viewing the pseudocode for main, I found
some cryptic code which I couldn't bother to pursue.

Back to the original file, for some clue. Somehow, I decided to do a `binwalk`
on the file and found a nice signature appended at the end:

    $ binwalk pure-luck.out
    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             ELF, 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux)
    273112        0x42AD8         Copyright string: "Copyright (C) 1996-2016 the UPX Team. All Rights Reserved. $"

Nice, some clue now: WTH is a UPX compression though? No matter, straight to
Google and found a [repo](https://upx.github.io/) to decompress the file. Now,
it seems like I've still got an executable, so `binwalk` again to see if there
are more data:

    $ binwalk pure-luck.out
    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             ELF, 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux)
    481360        0x75850         Unix path: /proc/sys/vm/overcommit_memory
    487717        0x77125         Unix path: /proc/sys/kernel/rtsig-max
    488794        0x7755A         Unix path: /sysdeps/unix/sysv/linux/getcwd.c
    490775        0x77D17         Unix path: /proc/sys/kernel/osrelease
    496388        0x79304         Unix path: /usr/lib/locale/locale-archive
    553468        0x871FC         ELF, 32-bit LSB no machine, (SYSV)
    557098        0x8802A         Unix path: /sysdeps/unix/sysv/linux/dl-origin.c

Okay, some stuff going on here, so back to IDA Pro for analysis. On decompiling
the main, I saw a bunch of variables which have static values:

    ...
    v10 = 102;
    v11 = 108;
    v12 = 97;
    v13 = 103;
    v14 = 123;
    v15 = 85;
    v16 = 80;
    v17 = 88;
    v18 = 95;
    v19 = 105;
    v20 = 115;
    v21 = 95;
    v22 = 115;
    v23 = 111;
    v24 = 95;
    v25 = 101;
    v26 = 97;
    v27 = 97;
    v28 = 97;
    v29 = 97;
    v30 = 115;
    v31 = 121;
    v32 = 121;
    v33 = 125;
    ...

Hey, this looks like ASCII! Let's use python to just convert them one by one:

    >>> a = [102,108,97,103,123,85,80,88,95,105,115,95,115,111,95,101,97,97,97,
             97,115,121,121,125]
    >>> print("".join(map(chr, a)))
    flag{UPX_is_so_eaaaasyy}

And done!

## Post Number (Medium, 100 pts, Open Source Cyber Intelligence)
I enjoyed this one, it was an internet-based goose-chase with real-world
implications. Anyway, we are asked to find a __post number__ (super misleading
word, if you ask me) based on an image. The image didn't ring any of my bells
but I know just the place to find original source images on the net:
[TinEye](https://www.tineye.com/).

Sorting by *Oldest* first returned a Flickr image by `ahmadmahmoudphotography`.
So let's copy the post number, 5410375058, and paste it into the answer box.
Nope, not it. Damn. Okay, could it be the post number assigned in the HTML? No,
the values change when I scroll onto different photos.

After a few hours trying other challenges, I gave the description a second look
and noticed that it mentions an address. At this point it clicked that it was
mentioning a house address so I browse the album to find
[this](https://goo.gl/xHRpCl) photo. And after putting in `11371` in the box,
I finally got it.

This has real-world implications as I've mentioned before: malicious people can
find your original address if you're not careful through this method. Suffice to
say this challenge creeped me out.

## Message in a Bottle (Easy, 50 pts, Digital Forensics)
I hated this one, simply because it was a prime example of me not seeing the
forest for the trees. Let's start. We get a PNG image, full details here:

    $ file message-in-bottle.png
    message-in-bottle.png: PNG image data, 350 x 144, 8-bit/color RGB, non-interlaced

Okay, that's fine, let's see if there is hidden information:

    $ binwalk message-in-bottle.png
    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             PNG image, 350 x 144, 8-bit/color RGB, non-interlaced
    41            0x29            Zlib compressed data, best compression

Nice! A Zlib compressed data, this must be what we're searching for. Let's
extract using `binwalk -e message-in-bottle.png`. Inside the folder is a file
called `29`. Interesting. Let's `strings` it... No good. What about the
`hexdump`? Nothing. Wow, okay. All this for 50 pts? Never matter.

Coming back after a while, I find that the values in the file are `0xff`
inflated. This is obviously a white background on some image! So I pulled up
python using `numpy` and `matplotlib` and reconstructed the image.

Time went by as I did other challenges and figured out proper dimensions for
the image and blah blah blah and ultimately got the image below:

Well, that's not helpful.



## Dark Project (Medium, 100 pts, Web Security)
TODO

## Get rid of them all (Easy, 50 pts, Malware Reverse Engineering)
TODO

## Hack the App (Hard, 200 pts, Malware Reverse Engineering)
TODO

## Intercept (Basic, 25 pts, General Information)
TODO

## They are many (Medium, 100 pts, Malware Reverse Engineering)
TODO

# CyberTalents CTF 2017 Write-ups
This is my second time doing CTFs so I thought I'd join the crowd and do a
write-up.

I'm using Ubuntu 16.04 for most of these terminal commands. I assume that
the commands are pre-installed on the system so I don't have to keep putting
in `sudo apt-get install <x>` everywhere. Time represented in the write-ups
are arbitrary. Since this was a take-home qualification round, a lot of time
was spent out with my family or doing something else. I took lots of break.

### Sections:

- [This is Sparta](https://github.com/syaffers/ct2017quals-write-ups#this-is-sparta)
- [Search in Trash](https://github.com/syaffers/ct2017quals-write-ups#search-in-trash)
- [Pure Luck](https://github.com/syaffers/ct2017quals-write-ups#pure-luck)
- [Post Number](https://github.com/syaffers/ct2017quals-write-ups#post-number)
- [Message in a Bottle](https://github.com/syaffers/ct2017quals-write-ups#message-in-a-bottle)
- [Intercept](https://github.com/syaffers/ct2017quals-write-ups#intercept)
- [Get rid of them all](https://github.com/syaffers/ct2017quals-write-ups#get-rid-of-them-all)


## This is Sparta

__(Easy, 50 pts, Web Security)__

The very first problem showed a website. Pretty innocent, if you ask me:

![This is Sparta?](https://github.com/syaffers/ct2017quals-write-ups/raw/master/sparta.png)

So, a username and password box. Trying `admin` and `admin123` returned wrong
user or password. Obviously, with these types of challenge, the first place to
check is the source. Oh look: a nicely mangled JS block. I took the script
out into my editor and prettified it:

    var _0xae5b = [
      "\x76\x61\x6C\x75\x65",
      "\x75\x73\x65\x72",
      "\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64",
      "\x70\x61\x73\x73",
      "\x43\x79\x62\x65\x72\x2d\x54\x61\x6c\x65\x6e\x74",
      "\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x43\x6F\x6E\x67\x72\x61\x74\x7A\x20\x0A\x0A",
      "\x77\x72\x6F\x6E\x67\x20\x50\x61\x73\x73\x77\x6F\x72\x64"
    ];

    function check() {
        var _0xeb80x2 = document[_0xae5b[2]](_0xae5b[1])[_0xae5b[0]];
        var _0xeb80x3 = document[_0xae5b[2]](_0xae5b[3])[_0xae5b[0]];
        if (_0xeb80x2 == _0xae5b[4] && _0xeb80x3 == _0xae5b[4]) {
            alert(_0xae5b[5]);
        } else {
            alert(_0xae5b[6]);
        }
    }

Nice, these cryptic hexadecimal values are just strings actually so let's
take a closer look at it in the console:

    > _0xae5b[0]
    "value"
    > _0xae5b[1]
    "user"
    > _0xae5b[2]
    "getElementById"
    > _0xae5b[3]
    "pass"
    > _0xae5b[4]
    "Cyber-Talent"
    > _0xae5b[5]
    "                      Congratz

    "
    > _0xae5b[6]
    "wrong password"

Okay, clearly it's checking the values of the username and password fields.
If we substitute the string values into the actual code, we should get the
following:

    function check() {
        var _0xeb80x2 = document["getElementById"]("user")["value"];
        var _0xeb80x3 = document["getElementById"]("pass")["value"];
        if (_0xeb80x2 == "Cyber-Talent" && _0xeb80x3 == "Cyber-Talent") {
            alert(_0xae5b[5]);
        } else {
            alert("wrong password");
        }
    }


According to the `if` statement, both of the fields need to equal 
`Cyber-Talent`. Okay, let's put that in and we get our flag.

![This is Sparta?](https://github.com/syaffers/ct2017quals-write-ups/raw/master/js_awesome.png)

`{J4V4_Scr1Pt_1S_Aw3s0me}` indeed.


## Search in Trash

__(Easy, 50 pts, Digital Forensics)__

We are given some file called `search-trash`, no extension. So let's `file`
it:

    $ file search-trash
    search-trash: Windows Recycle Bin INFO2 file (Win2k - WinXP)

Okay, so it's an INFO2 recycle bin file. I'm not too sure what it is but after
briefly looking at the file contents using `less`, there were several strings
which are quite contiguous but the remainder were mainly binary values. Let's
see if the flag is directly visible in the file using `strings`:

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


## Pure Luck

__(Basic, 25 pts, Malware Reverse Engineering)__

Again, we were presented with an unidentified file `pure-luck.out`. Upon
`file`-ing it, it's seems to be an executable, but is 32-bit ELF, which can't
be run on my 64-bit linux. No worries. Let's decompile it using IDA Pro. On
viewing the pseudocode for main, I found some involved code which I couldn't
be bothered to pursue.

Back to the original file for some clues. Somehow, I decided to do a `binwalk`
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


Okay, some more things going on here. I could've extracted the data but let's
just see if IDA Pro shows something new. On decompiling the main, I saw a
bunch of variables which have static values:

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

    >>> a = [102, 108, 97, 103, 123, 85, 80, 88, 95, 105, 115, 95, 115, 111, 95, 101, 97, 97, 97, 97, 115, 121, 121, 125]
    >>> print("".join(map(chr, a)))
    flag{UPX_is_so_eaaaasyy}


And done!


## Post Number

__(Medium, 100 pts, Open Source Cyber Intelligence)__

I enjoyed this one, it was an internet-based goose-chase with real-world
implications. Anyway, we are asked to find a __post number__ (super misleading
word, if you ask me) based on an image. The image didn't ring any bells but
the description did ask to find the original author of the image. I know just
the place: [TinEye](https://www.tineye.com/). This service allows you to find
the first instance of an image on the net.

By uploading the sample image and sorting by *oldest* first returned a
Flickr site with image by `ahmedmahmoudphotography`. So let's copy the post
number in the URL, 5410375058, and paste it into the answer box. Nope, not it.
Damn. Okay, could it be the image id number/tags assigned in the HTML? No,
the values change when I scroll onto different photos.

After a few hours trying other challenges, I gave the description a fresh look
and noticed that it mentions an address. At this point it clicked that the
problem mentioned looking for a physical house address of the guy so I browse
the album to find [this photo](https://goo.gl/xHRpCl). And after putting in
`11371` in the box, I finally got the flag.

This has real-world implications as I've mentioned before: malicious people
can find your original address if you're not careful through this method.
Suffice to say this challenge creeped me out.


## Message in a Bottle

__(Easy, 50 pts, Digital Forensics)__

I hated myself for this one, simply because it was a prime example of me not
seeing the forest for the trees. Let's start. We have a PNG image:

![The rabbit hole](https://github.com/syaffers/ct2017quals-write-ups/raw/master/message-in-bottle.png)

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

Coming back after a while, I find that the values in `29` are `0xff` inflated.
This is obviously a white background on some image! So I pulled up Python and
using `numpy` and `matplotlib`, I reconstructed the image. I figured out proper
dimensions for the image and blah blah blah and ultimately got the image below:

![Are you kidding me?](https://github.com/syaffers/ct2017quals-write-ups/raw/master/u_kidding_me.png)

After a moment pondering about self-reference (and my existence), I thought:
well, that's not helpful. It's got these black lines of different intensity
values which resembled the original image and didn't really provide any more
information. Okay, let's see if we can't analyze the intensity for some data.

__Time went waaay by. Like seriously, I wasted my time doing pointless
numerical analysis on this image__

Towards the deadline, I decided that I'm done with this, I'm just going to use
basic tools to see if I can get anything. And obviously, after shooting myself
in the face for so many times, this was indeed the right way to go. I tried
`steghide` but it's only for JPG. `stegsolve` was just too weird to use.
Desperately, I tried `openstego` and got the following error:

![I finally have you!](https://github.com/syaffers/ct2017quals-write-ups/raw/master/gotcha.png)

After trying an older version (`v0.6.1`), I got the flag `Flag{701_L@b$_CTF0}`.
Honestly, I've learned now that always try to try tools first before going off
the deep-end and over-analyzing. __Breadth-first, over depth-first__.


## Intercept

__(Basic, 25 pts, General Information)__

Oh this one pissed me the hell off. Like really, guys at CyberTalents, you
couldn't allow for multiple answers for questions like these? 😤

So the description asks for a type of attack where there is a person
intercepting data between two transacting parties. Obviously this is the
notorious man-in-the-middle attack. So let's put that in the answer. __RED__,
wrong!

What? How? Why not? So, I ended up trying (more than) a few, to no avail:

    man_in_the_middle
    maninthemiddle
    MANINTHEMIDDLE
    flag{man in the middle}
    flag{man_in_the_middle}
    flag{maninthemiddle}
    flag{MANINTHEMIDDLE}
    ...


You get the idea. So I thought, meh it's 25 points, who cares. Towards the
deadline, desperation ensues, I tried other possibilities like `interception`
or `sniffing` but I just had a hunch. I took one last stab with man in the
middle.

*Maybe this is it, maybe I can get it*. T minus 10 minutes.

    Man In The Middle

__RED__. Nope... T minus 9 minutes.

    elddim eht ni nam

__RED__. Yet again... T minus 8 minutes.

    man+in+the+middle

__RED__. Crap. T minus 7 minutes. Man in the middle... MITM.

    mitm

__GREEN__. My life was complete.


## Get rid of them all

__(Easy, 50 pts, Malware Reverse Engineering)__

This one is really nice but maybe, due to a stroke of luck, I got this
relatively quickly. So we're presented with a JAR file, which just spouts

    $ java -jar get-rid-of-them.jar
    No


Okay grumpy cat. Let's take you apart. I used
[this tool](http://www.javadecompilers.com/) to decompile the JAR and got two
files:

    $ ls -l
    -rw-rw-r-- 1 syafiq syafiq  834 Apr  7 21:04 Ctf.java
    -rw-rw-r-- 1 syafiq syafiq  948 Apr  8 10:45 ooo.java


Nothing suspicious, let's look inside `Ctf.java`. There is a line of code which
was particularly interesting:

    # Ctf.java
    ...
    static String flag = "&^&@|* Zm}&,);\\('))[\\[$`|_^#(x*]>&hZ)'$ $#(: [$3;&$t \\_']?&>,&i)!QG{`- ,% ~<`._@'::_\\_{}-|_[&{<`~$) ?'?(!$,.{>? @!^:#|R,?')`[,`;?!f_:$$<)Y}$:[|^?2)_h&><.:.-{&[|&A\\*;*)-($.>>(<^';#Q@?,,H\\`|)$ <):@(;}?-[~(&)>>*)(~)`$:[;>!.&%<!.>~ %J}*zX:(&:~:<0)*>(B(!?.#@A*<*{-,[Q@{%!~)~-~:@:#|![>)]?];H;$-<}>!@~)<<) \\_!|]#,&!,@>\\[]|J ]\\^[?>$|$?'|,#.)$l[^@X.~! \\;0-&,;,!['@[J*~#`AQ[*&%<,~]?~_^~(;}\\$>)[&@) (]}];;*^<)''@\\E[.@! B*.<-A-,:-#`-.}<-|)^Z@](?;H >-}.%.?}@<!())0] <&=@(<*$\\(("
    ...


Alright, this is interesting but seems like jumbled mess. Let's look further:

    # Ctf.java
    ...
    public static void main(String[] args) {
      if (args.length < 10) {
        System.out.println("No");
        return;
      }
      ooo o = new ooo();
      flag = o._2(flag, args);
      System.out.println(flag);
      System.out.println(o._1(flag));
    }
    ...


Okay, the `main` prints `No` if we try to run it with fewer than 10 arguments.
Otherwise it will print out the flag modified by this `ooo` object. Let's first
run the JAR with ten zeros:

    $ java -jar get-rid-of-them.jar 0 0 0 0 0 0 0 0 0 0
    &^&@|* Zm}&,);\('))[\[$`|_^#(x*]>&hZ)'$ $#(: [$3;&$t \_']?&>,&i)!QG{`- ,% ~<`._@'::_\_{}-|_[&{<`~$) ?'?(!$,.{>? @!^:#|R,?')`[,`;?!f_:$$<)Y}$:[|^?2)_h&><.:.-{&[|&A\*;*)-($.>>(<^';#Q@?,,H\`|)$ <):@(;}?-[~(&)>>*)(~)`$:[;>!.&%<!.>~ %J}*zX:(&:~:<0)*>(B(!?.#@A*<*{-,[Q@{%!~)~-~:@:#|![>)]?];H;$-<}>!@~)<<) \_!|]#,&!,@>\[]|J ]\^[?>$|$?'|,#.)$l[^@X.~! \;0-&,;,!['@[J*~#`AQ[*&%<,~]?~_^~(;}\$>)[&@) (]}];;*^<)''@\E[.@! B*.<-A-,:-#`-.}<-|)^Z@](?;H >-}.%.?}@<!())0] <&=@(<*$\((
    Wrong argsssss
    &^&@|* Zm}&,);\('))[\[$`|_^#(x*]>&hZ)'$ $#(: [$3;&$t \_']?&>,&i)!QG{`- ,% ~<`._@'::_\_{}-|_[&{<`~$) ?'?(!$,.{>? @!^:#|R,?')`[,`;?!f_:$$<)Y}$:[|^?2)_h&><.:.-{&[|&A\*;*)-($.>>(<^';#Q@?,,H\`|)$ <):@(;}?-[~(&)>>*)(~)`$:[;>!.&%<!.>~ %J}*zX:(&:~:<0)*>(B(!?.#@A*<*{-,[Q@{%!~)~-~:@:#|![>)]?];H;$-<}>!@~)<<) \_!|]#,&!,@>\[]|J ]\^[?>$|$?'|,#.)$l[^@X.~! \;0-&,;,!['@[J*~#`AQ[*&%<,~]?~_^~(;}\$>)[&@) (]}];;*^<)''@\E[.@! B*.<-A-,:-#`-.}<-|)^Z@](?;H >-}.%.?}@<!())0] <&=@(<*$\((


Okay, looks like there's more to this. Let's look into that `ooo`.java file.

    public String _1(String a) {
      try {
        return new String(Base64.getDecoder().decode(a));
      }
      catch (Exception e) {
        System.out.println("Wrong argsssss");
        System.out.println(a);
      }
      return "";
    }

    public String _2(String a, String[] b) {
      String temp = "";
      int i = 0;
      int j = 0;
      boolean bad = false;

      while (i != a.length()) {
        j = 0;
        bad = false;

        while (j != b.length) {
          if (a.charAt(i) == Integer.parseInt(b[j]))
            bad = true;
          j++;
        }

        if (!bad)
          temp = temp + a.charAt(i);
        i++;
      }
      return temp;
    }


So the `_1` function tries to decode a base 64 string which is the flag after
some processing by the `_2` function. I was too lazy to follow the `_2`
function thoroughly but based on the simplicity of the `_2` function (since
there are no values being edited, some some true/false flags), I thought
of just removing all the unnecessary characters from the original flag and 
leaving the base 64 characters only:

    >>> a = "&^&@|* Zm}&,);\\('))[\\[$`|_^#(x*]>&hZ)'$ $#(: [$3;&$t \\_']?&>,&i)!QG{`- ,% ~<`._@'::_\\_{}-|_[&{<`~$) ?'?(!$,.{>? @!^:#|R,?')`[,`;?!f_:$$<)Y}$:[|^?2)_h&><.:.-{&[|&A\\*;*)-($.>>(<^';#Q@?,,H\\`|)$ <):@(;}?-[~(&)>>*)(~)`$:[;>!.&%<!.>~ %J}*zX:(&:~:<0)*>(B(!?.#@A*<*{-,[Q@{%!~)~-~:@:#|![>)]?];H;$-<}>!@~)<<) \\_!|]#,&!,@>\\[]|J ]\\^[?>$|$?'|,#.)$l[^@X.~! \\;0-&,;,!['@[J*~#`AQ[*&%<,~]?~_^~(;}\\$>)[&@) (]}];;*^<)''@\\E[.@! B*.<-A-,:-#`-.}<-|)^Z@](?;H >-}.%.?}@<!())0] <&=@(<*$\\(("
    >>> b64only = list(filter(lambda x: x in "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=", a))
    >>> print(base64.b64decode("".join(b64only)))
    flag{b@d_ch@@rs_@@@re_B@@@@d}


And that was it. I guess my brain found a shortcut as it was scanning through
the `_2` code, can't really say how.

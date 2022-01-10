---
title: "Surviving the Quarantine"
date: 2020-05-13T15:26:00+01:00
categories:
 - misc
tags:
 - quarantine
 - cpp
 - cplusplus
 - programming
 - cobol
 - as400
 - ibm
 - ibm i
 - mainframe
 - midrange
 - thinkpad
 - windows
 - os2
 - rant
 - covid
---

> [2021-04-05:jack23247](jack23247@pm.me) In hindsight, this article might be a bit too cheeky, and it's useless: it was written by a bored and tired older me that'd been stuck home for a very long time. Read at your own risk.

Dear ghostly readers,<br>
I hope everyone's fine in these Coronavirus-laden times. As the quarantine struck pretty early in Italy, it was quite hard to find something to keep me entertained while staying at home and sitting at the computer screen all day.<br>
*Wait, isn't that what so-called computer "experts" always do?*

To keep my lazy self entertained, I've embarked on a couple never-ending projects as I always do. In no particular order I have:

- Started learning C++
- Dusted off my PUB400 account and wrote my first COBOL ILE program
- Restored the original IBM OEM Windows 95 on a ThinkPad 380D
- Been waiting for capacitors to arrive so I can fix the Xbox

So, since I'm too lazy to split it up, let's begin the **Quarantine Mega-Post Version 1 Release 1 Modification 0**. Yes, that's the whole name. Imagine having THAT on your ID card. Not funny.

## The ["better C"](http://www.stroustrup.com/C++.html)

This story started quite some months ago, when I decided to take the C++ Programming class, thinking that forcing me to learn the language was the only way to actually know it a little bit so I could sport it on my resumé. I absolutley **rejected** C++ initially: while I've always known that it was an important language to know (and eventually master), I was constantly set back by the fact that it was heavily oriented towards multi-paradigm and object-oriented programming. Besides being exposed to Java in first year's programming classes and kinda being a Ruby user, I just could not stand objects.

"Object are so lame and useless", I told myself and every fool who stood next to me, "they are *SO* heavy, It's just an artificial way of expressing computer programs. CPUs don't think like we do! See, C is a beautiful language. Its power stems from its simplicity, and it lets you control the hardware directly. Even the Linux kernel is written in C! This is what everyone should use for everything!".

Oh boy, I was so wrong.

Taking this dreaded C++ class turned out to be a life-changing experience after all, and it made me learn that, while C++ is NOT a better C, it's a very mature language and, paired with beautifully-crafted development tools, it's a damn powerhouse.

I also had to change my mind about object-oriented programming: it's a really nice way of doing things. While C++ objects may seem a bit weird at first, they provide the programmer with the flexibility needed to modify and adapt them to its programming style: unlike languages like Java and C#, the language does not force you to do things in a specific way. You can decide to call things however you want, nest namespaces in all kind of ways and you don't have to rely on rigidly-defined paradigms.

This flexibility has its limits though, and it's a good thing. In super-flexible interpreted languages like, say, Ruby you can redefine entire flocks of standard library functions and use them as needed. While indeed powerful, enabling this can (and will) limit your ability to go deep into the computer's bowels, as everything will be homogeneous and, essentially, very far from the underlying hardware.

> Indeed, this isn't always true: computers like the [LISP machines](https://www.cs.utah.edu/~mflatt/past-courses/cs6510/public_html/lispm.pdf) and [VLSI implementations of Prolog's WAM](http://xtf.lib.berkeley.edu/reports/TRWebData/accessPages/CSD-91-610.html) essentially expose a very different (and somewhat high-level) hardware paradigm, so you can really go deep (as far as mapping cons-cells to single real memory locations). It should also be noted that the proverbial *low-leveledness* of C is [debatable](https://queue.acm.org/detail.cfm?id=3212479), at least on modern computers. Assembly itself is [kinda fake](https://www.usenix.org/system/files/conference/usenixsecurity17/sec17-koppe.pdf), and x86_64 CPUs spend more die space to decode than execute μops. I hate it.

C++ is essentially the best of both worlds: it is compiled - so you'll get less runtime errors, it's very flexible - letting you express your programs the way you want and it's a superset of C - essentially letting you access hardware directly. And we haven't even talked about generics yet!<br>
*Spoiler*: we won't.

Ultimately, I think that C++ is an essential language to, at least, tackle for any programmer. Its paradigm mix-and-match approach lets you focus on what you think is important into any specific part of your program, and it helped me to think in a more rational way about programming languages paradigms and preferences.

What did I learn from this experience, then? That assembly is still the best. /s

## Is the AS/400 still relevant?

I've always had an itch for large systems: I've experimented countless times with Hercules, running either VM/CMS or MVS and dreamt about owning an IBM Mainframe. As much as I love the S/390s, I found myself more and more intrested in "Application System/400"s: I'm now the proud owner of two 400e Model 150s and a 400e Model 270. Sadly one of the 150s is a parts-only machine I got for free, and I'm afraid of turning on the 270 because of a disk problem (one disk broke so the RAID-5 is unprotected and I have no install media) which could render it a quite sexy brick.

![COBOL/ILE Test](https://raw.githubusercontent.com/jack23247/blog/master/img/pub400_cbllesrc_test.png)
*I eventually did it*

With the recent [rise](https://www.theverge.com/2020/4/14/21219561/coronavirus-pandemic-unemployment-systems-cobol-legacy-software-infrastructure) in COBOL programming needs, I thought I'd dust off my InfoWindow and start poking at the 400, to see if I could at least get my Hello World printed. Since the working 150 has no software installed at all, apart for the base system (which considering that OS/400 with ILE runtimes and DB/400 came preinstalled free-of-charge on the 150 it's one hell of a deal), if I wanted to dip my toes into mainframe-class business programming I had to put the literal green screen back in storage and find another way.

Suddenly it came to me that, while learning about how to start-up my 400s, I signed up for a free account on [PUB400.com](https://www.pub400.com/), which offers a free <del>OS/400</del> <del>i5/OS</del> IBM i learning environment.

![IBM i Main Screen](https://raw.githubusercontent.com/jack23247/blog/master/img/pub400_main.png)
*The IBM i Main Screen*

<!--https://raw.githubusercontent.com/jack23247/blog/master/img/-->

Why should one use such an environment?<br>
Because 400s are a fine piece of work and having a free playground is awesome.

OS/400 (I don't care what it's called by marketing) may seem a bit alien at first, but it's surprisingly easy to use and configure once you're up to speed: documentation is widely available and well-written, and it's a much leaner system compared to its mainframe ancestors: forget cryptic HLASM routines and weird JES2 commands: the system is mostly automated and CL is concise and streamlined. Commands have a logic, system-wide organization. For example:

```
WRKSYSSTS
```

is the CL command for "<u>W</u>o<u>rk</u> with <u>Sys</u>tem <u>St</u>atu<u>s</u>": most commands are named this way.

The whole system is organized in menus and you can use the `GO` command to jump from a menu to another. For example:

```
GO TCPADM
```

sends you to the TCP/IP admin options menu.

Function keys have special roles: `F3` quits a menu, `F4` brings up the completion pop-up menu and `F12` goes back one step. But most importantly, it has colors! The 5250 protocol supports a wide palette consisting of:
- green,
- white,
- black,
- blue,
- green,
- red,
- yellow,
- green,
- pink,
- cyan,
- green

and, uh, did I mention green?

Jokes aside, all of the menus features are crafted to be immediately recognizable with a consistent use of both color and glyphs, which guarantees that you'll feel at home in any of the system's many screens.

As far as programming goes, you have a nice REXX interpreter available and ILE, which is a catch-all software development platform with RPG, COBOL and C++ frontends to name a few, akin to Microsoft's CLR/.NET. Java (J9/WebSphere) is another notable option.

All of the development work is done via the Program Development Manager (PDM) and Source Entry Utility (SEU) combo, accessed via `STRPDM` and `STRSEU` commands, which I'll let you find the meaning for on your own, it's easy enough. PDM is basically a collection of menus that automate tasks like building and linking programs, and SEU is an XEDIT-like editor with advanced features. The downside is that PDM, SEU and the ILE development environment are an optional feature, as they're basically needed only on development boxes.

![he SEU Editor](https://raw.githubusercontent.com/jack23247/blog/master/img/pub400_seu_cobol.png)
*The SEU Editor*

Why did I tell you this much? To hopefully lower the entry barrier and let you at least think about trying OS/400. I could go on and on forever telling you about TIMI, the unique front panel codes, the flat memory space and object-oriented filesystem, about why the 400s are both fast on modest hardware and secure by design, and about the tight integration between hardware and software but I won't. This is what everyone keeps overstating about these systems and it's not that important for someone who just wants to give it a try. Besides I'm not qualified enough to do so, even if I intended to.

Please give the 400 a chance. It's for your own sake.

## The Internet Archive is a wonderful place

It truly is. Period.

I've been tinkering with my ThinkPad 380D for quite a while: it's a bulky and solid machine I absolutely love, but it (sadly) came without its original software. I thought I could install OS/2 Warp 4 quite easily on the beast but it turned out OS/2 is one weird beast and it doesn't like me (or maybe it's the other way around, really). I've tried putting Windows 98SE, NT 4.0 Workstation and even OpenStep 4.2 on the little beast and, to my surprise, they all worked fine. Yet I didn't feel satisfied, all of these gave off a solid but quite "stock" feeling that didn't satisfy my period-correctness obsession at all.

"Darn it", I thought, "let's look for a recovery disk."

![he SEU Editor](https://raw.githubusercontent.com/jack23247/blog/master/img/380d_recovery.png)
*Yes IBM HelpWare (R)PC Support Line, it's a new PC... I mean, I've never had one before*

And [there](https://archive.org/details/thinkpad380rec) it was: ready to be restored on this absolute unit of a laptop. The recovery disk works just fine, I just needed to dig out a Windows 95 OEM license from the og software closet: it even comes with old-school ThinkPad themes and ads! After waiting an eternity for the `1e-5`x CD-ROM drive (which is **not** bootable, btw) to finish reading the newly-pressed media, it was time to witness the glory of a machine cool enough to be used as an IBM System/390 Support Element. If only S/390s used ThiccPads as SEs instead of clunky beige IBM Personal Computer 750 boxes, which are quite sexy nonetheless.

> *Fun fact*: the battery still works and keeps the computer away from an outlet for about 1h30: not bad for a 23y/o cell.

## Where are my capacitors?

I have no idea, they're lost: my Xbox is in a sorry state and the damn Amazon seller won't contact me. I can't even go to one of the very few local suppliers because I'm stuck here because of the damn COVID laws.

Yuck.

> I contacted the local supplier and they had none, so I bought another batch of capacitors from AliExpress (of all places...) and after, like, six months they turned up! It took more than a year but I got my Xbox chooching away, even if the goddamn ground planes gave me one hell of a hard time, this summer (June 2021, this article was published in May 2020). 
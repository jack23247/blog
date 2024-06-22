---
title: "IBM 3151 Shenanigans!"
date: 2024-06-22T18:00:10+0200 ## date +%Y-%m-%dT%H:%M:%S%z
categories:
 - misc
tags:
 - retrocomputing
 - hardware
 - software
 - linux
 - freebsd
---

I recently got an IBM 3151 ASCII Display Station, which is a (somewhat) VT100-compatible terminal with a nice IBM look and feel to it. My unit has a green tube and closely matches the InfoWindow II Twinax terminals I'm used to, plus it uses a Model M keyboard instead of the crappy rubber-dome keyboards other manufacturers liked to use at the time.

Like all things IBM, however, the terminal is picky: you can't use it in 3-wire mode, as it requires signals such as Data Set Ready (DSR), Data Terminal Ready (DTR), and Data Carrier Detect (DCD) to work properly. 

In case you're curious, you can find the IBM 3151 [Reference Manual](
https://manx-docs.org/mirror/vt100.net/ibm/ibm3151_tr.pdf) and [Guide to Operations](
https://manx-docs.org/mirror/vt100.net/ibm/ibm3151_gto.pdf), courtesy of `manx-docs.org`.

## Configuring the 3151 for 3-wire communications

### Building the loopback adapter

Luckily, someone had already stumbled across the problem, and the awesome guys at the VCF forums found original cables and traced them, devising a [solution](
https://forum.vcfed.org/index.php?threads/ibm-3151-serial-terminal.63591/).

First of all, we need to build a loopback adapter to check if the terminal works. Using a female DB25 connector, it can be done as follows:
- Connect pins #8 (DCD), #6 (DSR), and #20 (DTR) together;
- Connect pins #5 (CTS) and #4 (RTS) together;
- Connect pins #3 (RX) and #2 (TX) together.

<img src="https://raw.githubusercontent.com/jack23247/blog/master/img/3151_adapt.png" alt="loopback-adapter" style="width: 21em;" />

> If you need a refresher on which pin does what, look no further than [Lammert Bies' guide](https://www.lammertbies.nl/comm/cable/rs-232), which is the best introduction to RS232 I found.

### Configuring the terminal

Power on your terminal and enter the configuration menu using the *Ctrl-Setup* key combination (setup is the blank key in the top-right corner of the numpad) and make sure it is configured as follows:

- **General**
  - Machine Mode: **IBM 3151**
  - Row and Column: **24x80**
  - Auto LF: **OFF**
  
- **Communication**
  - Operating Mode: **ECHO**
  - Line Speed (bps): **9600**
  - Word Length (bits): **8**
  - Parity: **NO**
  - Stop Bit: **1**
  - Line Control: **IPRTS**

You can switch between tabs using the *Send* key and choose an option using the spacebar. Once you're done, go to the **Function** section, save, and press *Ctrl-Setup* to exit.

If you press a key on the terminal, you should see nothing appear on the screen: this means that the terminal is configured correctly for 3-wire operation. Now, if you connect the contraption to the rightmost DB25 port on the back, you should see characters reflected back through the loopback adapter as you type.

### Modifying the adapter

Once you've verified that the terminal works, you can modify the adapter to let you attach essentially anything that talks via serial through a regular DB9 cable.

Grab a female DB9 connector and attach it as follows:
- Connect pin #7 on the DB25 plug to pin #5 on the DB9 plug (Data Ground);
- Connect pin #3 on the DB25 plug to pin #3 on the DB9 plug (Receive Data);
- Connect pin #2 on the DB25 plug to pin #2 on the DB9 plug (Transmit Data).

<img src="https://raw.githubusercontent.com/jack23247/blog/master/img/3151_adapt_2.png" alt="db9-adapter" style="width: 36em;" />

If all went well, you should now be able to use a straight-through DB9 cable to attach the terminal to an USB UART or a real serial port on any computer, as long as the parameters match.

> Once you've verified that everything works, you can switch to a faster speed such as 38400 Baud (the fastest this terminal supports). Keep in mind that 9600 is a "safe speed", and many devices default to it.

## Connecting to a modern system

While modern computer systems don't expect to be connected to a serial terminal at all times (it's not the 80s anymore, maybe you haven't got the memo), the fundamental concepts of UNIX character devices has not changed at all... and a serial terminal is the _quintessential_ character device!

I'll try (and, spoiler, somewhat succeed) in connecting the IBM 3151 to a modern computer, in order to use modern, top-notch software such as `top`, `sh`, `vi` and `ls` from a properly retro computing device, as momma IBM intended. Jokes aside, using a terminal is a somewhat cathartic experience[^1], as you do away with distractions and other modern amenities such as graphics that clutter your field of vision and distract you from the task at hand.

[^1]: Unless you use `emacs-nox`, you vile monster!

### Connecting to a Linux system

Conencting to a Linux system should be easy, but it's not. 

> First of all, if you have SeLinux, nothing will work properly unless you issue `setenforce 0`... Fedora users, I'm looking at you!

I used an USB UART, which we'll call `/dev/ttyUSB0`. Once you have connected the terminal to the adapter, you can try writing to it (as root, or you can `chown` the device):

```
linux# echo "It works!" > /dev/ttyUSB0
```

If nothing appears on the terminal, you have an issue. Check `dmesg` first (e.g. one of my adapters was flakey and the driver was very angry with it), then your cabling: make sure you're using a straight cable and not a nullmodem, for example.

If gibberish appears instead, you're on the right track! Communications are established, but the parameters are incorrect. You can use `stty` to see and change the configuration:

```
linux# stty -F /dev/ttyUSB0
speed 115200 baud; line = 0;
-brkint -imaxbel
```

If the speed is wrong, for example, you can change it as follows:

```
linux# stty -F /dev/ttyUSB0 speed 9600
9600
linux# stty -F /dev/ttyUSB0
speed 9600 baud; line = 0;
-brkint -imaxbel
```

Try `echo`ing something to the serial port now. If it works, try `cat /dev/ttyUSB0` and press some keys on the terminal's keyboard: you should see them appear both here and on the screen.

Now that you've verified that bi-directional communication works, you're ready to connect using `agetty`.

```
linux# agetty -n -l $(which bash) ttyUSB0 38400
```

Now `bash` should open on the terminal, but you'll see a lot of gibberish: this is because `bash` expects you to be using a modern terminal emulator, not some ancient piece of crap. We can partially fix that by issuing `export TERM=vt100`, but since the IBM 3151 is not 100% VT100-compatible, everything will be broken.

The IBM 3151 and uses its own Terminfo entry, aptly called `ibm3151`; modern Linux distributions, however, seem to lack most entries related to old (real) terminals which are not used anymore. For example, on my system there's a lot of `xterm` entries, but no `ibm3151`:

```
linux# find /usr/share/terminfo -name vt100
/usr/share/terminfo/v/vt100
linux# find /usr/share/terminfo -name xterm-*color
/usr/share/terminfo/x/xterm-color
/usr/share/terminfo/x/xterm-16color
/usr/share/terminfo/x/xterm-256color
/usr/share/terminfo/x/xterm-88color
/usr/share/terminfo/x/xterm-pcolor
linux# find /usr/share/terminfo -name ibm3151
```

Literally to no one's surprise, [an arch user on /r/unixporn](https://www.reddit.com/r/unixporn/comments/fgn95v/ibm_3151_bash_minimalism_is_for_the_weak/) found a solution: the utility used to build `terminfo` files, called `tic`, is available and can be used to build missing entries from sources; they also provided a link to [the most recent terminfo sources](http://invisible-island.net/datafiles/current/terminfo.src.gz), but omitted the exact parameters you need to invoke `tic` with.

After a bit of fiddling, I figured it out:

```
linux# wget http://invisible-island.net/datafiles/current/terminfo.src.gz
linux# gzip -d terminfo.src.gz
linux# tic -e ibm3151 -o /usr/share/terminfo/ terminfo.src
"terminfo.src", line 474, col 11, terminal 'ecma+color': unknown capability 'AX'
"terminfo.src", line 486, col 20, terminal 'ecma+strikeout': unknown capability 'rmxx'
"terminfo.src", line 486, col 32, terminal 'ecma+strikeout': unknown capability 'smxx'
<...>
linux# find /usr/share/terminfo -name ibm3151
/usr/share/terminfo/i/ibm3151
```

You can safely ignore all of `tic`'s complaints, as they don't affect `ibm3151`. Now you should be able to `export TERM=ibm3151` and use goodies such as `clear` and `nmon` from your terminal!

<img src="https://raw.githubusercontent.com/jack23247/blog/master/img/ibm3151.jpg" alt="ibm3151" style="width: 35em;" />

Here you can see the 3151 running `top` in all of its glory.

As for getting a login shell going, I followed [the guide](https://0pointer.de/blog/projects/serial-console.html) and got a prompt via systemd, but could never figure out how to login properly, so I gave up; this is weird, since I clearly remember logging into my laptop via DOS Kermit.

### Connecting to a FreeBSD system

FreeBSD is embarassingly simple in comparison, you just need to modify `/etc/ttys` to specify your terminal's parameters and reload the `init` daemon with `kill -HUP 1`; of course, I got all this from the [handbook](https://docs.freebsd.org/en/books/handbook/serialcomms/#term-config), no googling required.

>  Oh and BSD still has a Terminfo entry for the IBM 3151, so there's no need to build anything from source. Yeah, sometimes Linux sucks.

You can also tell the BSD loader that you want to use the terminal as a boot console, and you'll be able to see all of `dmesg`'s chatter.
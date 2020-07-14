---
layout: post
title: "rgbCTF 2020 Writeups"
categories: ctf
---

## Alien Transmission 1 {#alien-transmission-1}
### Category: Forensics/OSINT | Solves: 30 | Points: 219

{:refdef: style="text-align: center;"}
![Alien Transmission Challenge Text](https://i.imgur.com/EIH4MFK.png){: .imgCenter}
{: refdef}

We're given a .WAV file, and the clue tells us that it came over the radio, so right away we can go looking for ways to decode this 36 second audio clip into something meaningful.  When you open it with Audacity and look at the spectrogram,  we are presented with the image on the left.  After a little bit of digging, the image matches pretty closely with what a [Slow Scan Television (SSTV)](https://en.wikipedia.org/wiki/Slow-scan_television) signal looks like (image on the right) 

{:refdef: style="text-align: center;"}
![Audacity ](https://i.imgur.com/0dOxkYh.png){: .imgCenter}
{: refdef}

Often times programs that pull data from audio involve complex setups with virtual audio devices and whatnot. Luckily there's a tool on GitHub ([xdsopl/robot36](https://github.com/xdsopl/robot36)) to encode/decode SSTV in a mode called Robot36. After compiling the tool, we run it against the file and grab our flag!

{:refdef: style="text-align: center;"}
![robot36 output](https://i.imgur.com/HqFVEe8.png){: .imgCenter}
{: refdef}

{:refdef: style="text-align: center;"}
![The Flag](https://i.imgur.com/fDnVkar.png){: .imgCenter}
{: refdef}

## Adventure {#adventure}
### Category: Misc | Solves: 21 | Points: 495

{:refdef: style="text-align: center;"}
![Adventure challenge description](https://i.imgur.com/jHrTIPk.png){: .imgCenter}
{: refdef}

Right off the bat, there's a huge hint in the description with the weird capitalization that spells out ATARI.  Okay, so we can check to see if this is actually an Atari game by throwing it in [an emulator](https://stella-emu.github.io/downloads.html).  We pop it in and lo and behold, we have ourselves a game!

{:refdef: style="text-align: center;"}
![Breakfast game for Atari](https://i.imgur.com/PfStxS5.png){: .imgCenter}
{: refdef}

So what is immediately obvious is that this is not a standard game (I mean your character sprite is just the number 1.)  Any time there are customized/edited assets inside a game, I try to "work backwards" and think of what sort of tools are out there to do this specific thing.  After a bit of searching, I found [Hack-O-Matic III](https://www.romhacking.net/utilities/723/) on [Romhacking.net](https://www.romhacking.net). It's a simple program that shows the ROM literally bit-by-bit, making it easier to spot sprites.  Sure enough, that leads us to the flag. 

{:refdef: style="text-align: center;"}
![The flag](https://i.imgur.com/Q3n00SO.png){: .imgCenter}
{: refdef}



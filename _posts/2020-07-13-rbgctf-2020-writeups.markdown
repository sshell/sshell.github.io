---
layout: post
title: "rgbCTF 2020 Writeups"
categories: ctf
---

## Alien Transmission 1 {#alien-transmission-1}
#### Category: Forensics/OSINT | Solves: 158 | Points: 219

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
#### Category: Misc | Solves: 21 | Points: 495

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

## Robin's Reddit Password {#robins-reddit-password}
####  Category: Forensics/OSINT | Solves: 30 | Points: 490

{:refdef: style="text-align: center;"}
![Challenge text](https://i.imgur.com/TGz3N9X.png){: .imgCenter}
{: refdef}

I have to admit that at first I couldn't quite wrap my head around this one, but the bit about breaking into Reddit's server reminded me of a funny easter egg someone had sent me before.

{:refdef: style="text-align: center;"}
![reddit easter egg](https://i.imgur.com/n7Fe1SN.png){: .imgCenter}
{: refdef}

Sure enough, one of the users here is Robin.  We google the hash and find out that someone has already cracked the hashes for us (thank you, stranger) so we can just wrap it in the flag format and submit it!

{:refdef: style="text-align: center;"}
![reddit hashes crakced](https://i.imgur.com/TZF5DWe.png){: .imgCenter}
{: refdef}

It's also worth noting that this challenge could have just as easily been solved with one Google search as well.

{:refdef: style="text-align: center;"}
![](https://i.imgur.com/KxCvY13.png){: .imgCenter}
{: refdef}

## PI 1: Magic in the Air {#magic-in-the-air}
####  Category: Forensics/OSINT | Solves: 52 | Points: 470

{:refdef: style="text-align: center;"}
![Challenge text](https://i.imgur.com/6EMB1Ud.png){: .imgCenter}
{: refdef}

Right off the bat we know we're dealing with some sort of sniffed wireless traffic. Opening the unzipped file in a hex editor confirms that we're dealing with a Bluetooth capture, so we can go ahead and open it up in Wireshark and get a better idea of what exactly we're working with.

{:refdef: style="text-align: center;"}
![File in hex editor](https://i.imgur.com/OzIlchO.png){: .imgCenter}
{: refdef}

{:refdef: style="text-align: center;"}
![Bluetooth capture in Wireshark](https://i.imgur.com/SDHijwo.png){: .imgCenter}
{: refdef}

The source is named G613 and we know it's a Human Interface Device, a quick Google search will show that it's a [wireless keyboard](https://www.logitechg.com/en-us/products/gaming-keyboards/g613-wireless-mechanical-gaming-keyboard.html). A quick once-over of the file reveals that the most commonly received message is this "Rcvd Handle Value Notification" which has the same 12-byte header and a constantly changing 13th byte. This is easily confirmed by flipping through packets while referencing [this keymap](https://github.com/greatscottgadgets/libbtbb/blob/master/python/pcaptools/btaptap) from the [greatscottgadgets/libbtbb](greatscottgadgets/libbtbb) repo on GitHub. We convert the file to hex and then write this snippet of code to regex out the information we need and compare it to the keymap dictionary we found earlier.

{:refdef: style="text-align: center;"}
![Python code to extract keystrokes from Bluetooth capture](https://i.imgur.com/G4W4aCA.png){: .imgCenter}
{: refdef}

The result provides us with the phone number and country it's from. A quick Google search reveals that +46 is the dialing code for Sweden, so we have everything we need to solve this challenge and start on the next!

{:refdef: style="text-align: center;"}
![Extracted keystrokes](https://i.imgur.com/r0pCHXC.png){: .imgCenter}
{: refdef}

## PI 2: A Series of Tubes {#a-series-of-tubes}
####  Category: Forensics/OSINT | Solves: 22 | Points: 495

Coming Soon

## Typeracer {#typeracer}
####  Category: Web | Solves: 184 | Points: 119

Coming Soon
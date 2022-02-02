---
author:
  name: "Or0ch1"
linktitle: Knight CTF 2022
title: "Knight CTF 2022"
categories: ["CTF"]
tags: ['CTF', 'Forensics', 'Steganogrphy', 'Python']
type:
- post
- posts
date: 2022-02-02T08:56:51+03:00
---
![Logo](/static/knightctf/knight_ctf_logo.png)

This was one of the CTFs that started off the year and it was an online jeopardy style Capture the Flag (CTF) competition hosted by the Knight Squad community from Bangladesh.

The CTF started on 20 January 2022 09:00PM GMT +6 and ended on 21 January 2022 11:59PM GMT +6.

I participated under team [@Fr334aks-mini](https://twitter.com/fr334aksmini) and we managed pos 120 out of 752 teams with 2750 points.

![Team_Position]({{<static "/knightctf/position.png">}})

I focused primarily on the forensic and steganography categories and managed to solve 4 challenges,2 from each category.

## Forensics
### Lets walk Together
Find the flag from the following: [Download Link](https://drive.google.com/drive/folders/1wjBWBjz2e_DxgSNvD-j5vXoKahZiVfLR).

First thing i always do whenever i get an image is to run file and strings on the image file in order to check that file extension matches the file type and to check to see if any hints appear from the strings in the file.
In this case the output from strings showed a flag.txt hinting towards a file being embedded in the image.

![chall]({{<static "knightctf/forenchll4.png">}})

I decided to use foremost to extract the file from the image and got a zip file.

![foremost]({{<static "knightctf/foremost.png">}})

![ls]({{<static "knightctf/lsout.png">}})

Tried unzipping the file but it needed a password,i cracked it using fcrackzip and the rockyou.txt wordlist got the password and extracted the file and got the flag.

![crack]({{<static "knightctf/crack2.png">}})

## Unknown File
Find the flag from the following: [Download Link](https://drive.google.com/drive/folders/1bNDZ_csIPx_hG0_ou1WgL884aEpMajSe)

As i always do i ran file on the file and it ws identified as data. That stumped me for a while till i decided to view it in hexdump and noticed the file had IHDR and IEND chunks so i knew that it was an image.

![ihdr]({{<static "knightctf/IHDR.png">}})
![iend]({{<static "knightctf/IEND.png">}})

On investigating the IEND chunk further i noticed the end chunk value **49 45 4E 44 AE 42 60 82** was that of a png image.

![iendhex]({{<static "knightctf/iendhex.png">}})

I researched a bit about fixing png headers and found this tool [PCRT](https://github.com/sherlly/PCRT.git) that automatically fixes the headers.I added a png extension to the file and the tool automatically fixed it.

![pcrt]({{<static "knightctf/PCRT.png">}})

![output2]({{<static "knightctf/output2.png">}})



## Steganography

### FileD
Find the Flag from the following: [Download Link](https://drive.google.com/drive/folders/1tZi4QQhxGWJBIQB62-1zjU5jiGG97OQG)

On downloading you get a file with a .kra extension,i have never seen such an extension before so i immediately googled about it and found this site [KRA File](https://fileinfo.com/extension/kra) which explained that the file was an image created by the application Krita Image viewer.

It is provided as an appimage, hence you just download it give it execution permissions and run it.

![krita]({{<static "knightctf/krita.png">}})

Opening the image using the application showed the image itself and in a separate pane showed the layers that the image had.

![layers]({{<static "knightctf/layers.png">}})

I went about deleting each layer till i got to the flag.

![rightlayers]({{<static "knightctf/rightlayer.png">}})
![flag]({{<static "knightctf/flag.png">}})


### Follow The White Rabbit
Find the Flag from the following: [Download Link](https://drive.google.com/drive/folders/1NSPLcRnbhdwYdaNttpvidJ2RiaPxk0rR)

We get an image with a rabbit, however when you look at it closer you see a series of dashes and fullstops. 

![whiterabbit]({{<static "knightctf/whiterabbit.jpg">}})

I immediately knew it was morse code and went about decoding it with [Morse Decoder](https://morsedecoder.com/).

![morse]({{<static "knightctf/morse.png">}})

The decoded text was to be placed in **KCTF{}** and submitted as the flag.


### QR Code From The Future
Challenge file can be found here: [Download Link](https://drive.google.com/drive/folders/1TDizwDbBhBR1bz9LYYqBrDL6cUtrMqEV)

I did solve this challenge but not before the flag was submitted by the team captain Winter.

This challenge really gve me a headache,it was a git that contained qrcodes. 

I ran strings,binwalk, foremost and all i hit were dead ends till i decided to stop thinking along the lines of "There must be a file or text embedded in it".

Since it was a gif but contained qrcode i decided to scan it but each and every time i scanned the qrcode i got a different letter which was when i realized these were multiple qrcodes in the gif.

I researched a bit and found a site which extracted each frame and saved them as images in a zip making it easier to work with the qrcodes [EZgif](https://ezgif.com/split). 

![morse]({{<static "knightctf/gif.png">}})

![morse]({{<static "knightctf/zip.png">}})

I now had a new problem, i had 48 different images and i shuddered to think that i was going to have to manually load them into a qrcode scanner so i created 2 python scripts, one to decode the qrcodes in bulk and output the text and the second to format the text onto a single line.

The scripts can be found on my github [Or0ch1](https://github.com/Or0ch1/Scripts)  

![morse]({{<static "knightctf/pyscript.png">}})

One huge problem i faced was how the program was reading the files randomly so for an hour i was stuck trying to decode text that was in the wrong order.I fixed this by using the nasorted python library.

The text gotten was shifted using ROT13 and it was also reversed so i used this online decoder [ROT13](https://rot13.com/) and got the flag.

![morse]({{<static "knightctf/Rot.png">}})
![morse]({{<static "knightctf/code.png">}})



I really hope you enjoyed the read, as im sure most can tell this is my first time writing a writeup :joy: but i hope to get better with time.

Happy hacking everyone :sunglasses: and as we say at f334aks #IWDWD :skull:

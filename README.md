# commodore-64-koala-picture-linker
A guide on how to link a Koala Pad picture to another program

The purpose of this tutorial is to link a Commodore 64 Koala picture with a secondary program such as a game. 

How to link a Koala picture to a program

1) If your Koala picture is already compressed (you can load it and type RUN and it displays) go to step 6.

2) Download Koala Cruncher
https://csdb.dk/release/?id=107958

https://csdb.dk/gfx/releases/107000/107958.png

3) Load the cruncher and pack your Koala picture. In this example we're going to use the Koala Paint Slideshow disk for our source images.

https://csdb.dk/release/download.php?id=170435

https://i.ibb.co/NL49YFf/d1.jpg

4) Enter in the filename of the uncompressed Koala file (40 blocks in size). In this example we'll use the "HEAVY" file. This is the fourth file from the top. If the file load is successful you'll see the disk message:
00, OK, 00, 00. You only need to enter the filename, not the "PIC A " part.

https://i.ibb.co/jyJqDPG/d1.jpg


5) As you can see, I've saved the file under the name "test".

https://i.ibb.co/Xy29K84/ww.jpg

This is the image we've chosen:

https://i.ibb.co/gj1y0zw/Image5.jpg

6) We now need to find a way to link the picture to our program. 

If you look at the 64's memory from a visual point of view:

https://i.ibb.co/k1kMK20/mik.jpg

Picture the GREEN bar as the entire RAM memory in the C64. The beginning of Basic memory is as the bottom ($0801), the end of the memory ($9FFF) is at the top. Every program we ever load and type RUN, will load into the GREEN area. Larger programs will use more of the green area, smaller programs will use less area.

In the case of our "TEST.PRG" file, the file is quite small and doesn't use all of the Basic memory. This is why it's not using all of the green area.

A fictional game of 50 blocks size is using the RED area.

We can load and run the TEST.PRG picture or we can load and run the 50-block game, but we can't have both in memory at the same time because they both occupy the SAME memory. In order to run both, we'd have to reset the 64 and load the other file.

In order to fix this, we have to place the game in memory AFTER the picture. This way when the picture is viewed and you press a key, the game is run.

https://i.ibb.co/B2FZfLs/Image4.jpg

Have you ever loaded a cracked game with a crack intro? When you press the space bar the screen border flashes, there might be some static noises, and there's a mix of different characters on the screen. Some of the screen characters might also be rapidly changing.

This is a memory transfer routine and its purpose is to copy the RED area down into the YELLOW area. Because you can't have two programs in the same memory area at once. Get it? :D The YELLOW program is overwritten as it's served its purpose. The RED program takes over that memory and is run.

The reason that you see the transfer taking place on the screen is because the screen is a 'safe' area. If you put the transfer routine into the actual intro, it would be overwritten when the RED was copied into the YELLOW.

By this same logic, we cannot have a memory transfer routine in the actual Koala picture file. If we did, it would be overwritten when we pressed Space and before the transfer was finished.

It's not difficult to fix this but there's a complication. That BLUE area of memory. As you can see, it's not being used by either the Koala picture or the game. But it is RESERVED because when we run the Koala picture, that area gets used.

https://i.ibb.co/0VkcjMB/ff.jpg


7) We're going to have to make some adjustments to put this all together. We don't need to make any changes to the Koala program because it exits to Basic when you press a key. So we can call it from another program without issue.

We do need a transfer routine and that routine, by no coincidence, will be stored in the screen memory at $0400 (1024).

This is our commented routine:

```
JSR $080D  ; call the Koala picture
SEI  ; stop what you're doing IRQ
LDA #$34 ; turn off ROM so we can access the full 64K memory
STA $01   ;
LDX #$00 ; set X to zero
A LDA $8000,x
STA $0801,x  ; load $8000 and put it into $0801
INX  ; increment x by 1
BNE A  ; loop until x is 0 again (256 bytes copied)
INC $040C  ; change $8000 to $8100
INC $040F ; change $0801 to $0900
LDA $040C
BNE A    ; copy 256 bytes then move to the next bank and check
           ; to see if $8000, $8100, $8200, etc. has reached $0000
           ; and theres nothing more to move down into $0801
LDA #$37
STA $01  ; turn ROM back on again
CLI   ; thanks for the time IRQ
JSR $A659
JMP $A7AE  ; perform RUN
```

The program works as follows:
It sets X to zero and copies $8000,x to $0801,x. This moves a single byte at $8000 into $0801 and is the first part of our copy routine. Think of the ,x as "+x"

We then increment X by 1 so that we copy $8001 into $0802, and when we increment X again it will copy $8002 into $0803. We increment X until it reaches 0 again (wrapping back to zero). The BNE A branches back if X is BNE (branch not equal to zero).

Now we increment the $8000 to $8100 and the $0801 to $0901. This is the rapidly changing characters you see on the screen after a crack intro - the incrementing of the banks being copied. $040C and $040F point to the $80 and $08 on the screen.

The
LDA $040C
BNE A
checks to see if the $8000 has moved up to $FF00. It does this because when the $FF  in $FF00 is incremented, it wraps to $0000. If we didn't do this, the memory transfer routine wouldn't know when or where to stop.

We then turn the Basic back on by putting #$37 into $01 and we perform a "RUN" with the JSR $A659 and JMP $A7AE.

You can download the transfer routine and all necessary files here:

https://github.com/milasoft64/commodore-64-koala-picture-linker

7) Okay that's it.... now we compile the three files into one.

Download ECA Linker
https://csdb.dk/release/download.php?id=19494


Press a key to enter the main program. You'll be asked for the Low-Mem. This is the lowest point in memory that you'll be using.

In this case because we're using a screen-based transfer routine, we're going to enter $0400 because we want to keep the screen memory preserved when ECA unpacks.

https://i.ibb.co/kJGrkXN/eca2.jpg

Press return for the Skip$. I've never gotten a proper explanation of what this does but I believe it's meant to skip a memory bank area such as $D000.

Now you can enter your files or "$" for a directory:

https://i.ibb.co/DbP2G26/eca.jpg

It doesn't matter the order that you enter them:
XFER  (the transfer routine I made)
PAC MAN ARCADE (the game to be linked to the picture)
TEST (the compressed Koala)
just do enter them one by one.

https://i.ibb.co/BgJpv04/ee.jpg

When it comes to the second program, the game you want linked to the photo you will add ",8000" to the filename. This will link the game up above the Koala pic and reserved memory ($8000).

You're done.

Enter @ for a filename to signal that you're done. Wait for the program to crunch the files.

https://i.ibb.co/wKYSzfN/ecalinked.jpg

Finally the SYS-ADR$ is the address we want to run when the files are decompressed. If we entered $080D it would show the Koala. We will enter $0400 instead which will call the Koala picture AND transfer the Pac Man game for us.


Milasoft

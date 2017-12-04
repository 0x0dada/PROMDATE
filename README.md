This repo is based upon a fork (which has been reset to commit
150223f2b7c302cb04e4cfbb7c424cfa99576c01) of Trammel Hudson's et. al
excellent PROMdate project.

I'm not a fan of Arduino IDE, and the original project was ported to Arduino 
in the first quarter of 2017 (see git log of the original repo @
https://github.com/osresearch/prom ).

If you're happy with the Arduino IDE, use the original flavor.

# Why

Being neighborly is a good thing.

As mentioned above, the latest flavor of PROMdate didn't work for me for
aesthetic reasons.  That's just me, and nothing against the fine folks
at Arduino or Mr. Hudson et al.

More importantly, there's little documentation for the original project, 
beyond a few photos and the code itself.  As time permits I'll try to rectify 
that here.  At some point Mr. Hudson may get around to crufting together
some documentation, but in the mean time, if you need a path to follow,
this will hopefully help.

Most of the following instructions apply to generic flavors of Linux 
(Debian etc.) OpenBSD requires some tweaking; eventually I'll post
OpenBSD specific instructions here.  It's most likely that anybody
coming to this repo will be a Linux user.  If you're hacking hardware
with OpenBSD, you tend to know what you're doing. ;)  This is nothing
against Linux!  I use both, all the time, for different reasons.  I
preach agnosticism -- use what you have wherever it works, use something
else if it doesn't, try to figure out why it was failing for extra
points.

# BOM

* 1 Teensy++ (Buy from a real distributor, not Amazon.)
* 4 Short lengths of jumper wire
* 1 40 pin ZIF socket
* 1 5V power source (PL2303 is a cheap source. http://dipmicro.com)

Everything else: distributor of your choice (Adafruit, Mouser etc)

## Assembling

First things first.  Plug your Teensy++ into your computer. It has been
pre-programmed with  the canonical "blinky" program.  LED flashing?
Good, your board is probably fine.  Check to see that it enumerates.
Gasp! lsusb shows ... nada...?!?  Remain calm.  Push the little reset
switch.  lsusb will now show the device enumerating with the HalfKay
bootloader.  Great, the board works.  If you think that skipping this
basic check is a good idea, you have not made it around the block even
once.  Don't worry, a few more muggings and you'll be a veteran.

Orient the Teensy++ on the pin side of the ZIF socket with the USB
connector toward the latch of the ZIF socket. Place the Teensy++ onto
the ZIF socket, *leaving the bottom two pins of the ZIF socket exposed
and 5V and GND holes on the Teensy++ disconnected.* Everything fit?
Good.

*Leaving the Teensy properly placed on the ZIF socket* carefully mark
the surface of the ZIF socket next to the GND and AREF pins.  See the
nifty little pocket card that came with your Teensy++ if you prefer not 
to squint at the board.  Remove the Teensy++.

You now have two options:

Option A) *Carefully* bend or bend and twist the two marked pins
*outward*.  The bend needs to be down low enough for the board to fit
over the other pins for soldering and *not make contact* with the two 
bent pins. Solder two of your jumper wires to these pins.  Bending 
delicate little pins unnerves me, I went with:

Option B)  Solder two of your jumper wires, one on each marked pin of
the ZIF socket, way down low, as close as possible to the surface of socket 
without melting the silly thing.  I bent the exposed part of the wire
into a tiny loop (my fear does not extend to wire) to help hold it
during the soldering process. Solder them up.  If you've been careful,
after soldering you'll have half of the original pins sticking
up.  Next, cut out a piece of *antistatic bag* a bit larger than the ZIF
socket.  Poke the pins through. You'll need to make slightly larger
holes for the soldered pins. This is a *dust bib*. Nip the bit of the
exposed pins off that's standing proud of the solder, and then taking a fine 
file or emery board, *gently* remove the rest of the soldered pin and
extra solder material until you can place the Teensy++ onto the ZIF
socket with the rest of the ZIF pins barely poking through the board and a tiny 
air gap between AREF and GND.  Remove the Teensy++ after the final check and
clearance confirmation, and place two thin strips of electrical tape
over AREF and GND separately and a slightly larger strip over both. (You
can  use other insulating materials, this is what I had on hand.)

Next, locate the PA3 and PA7 holes on the Teensy++. You're going to
poke the two remaining jumpers into these holes from the top (chip/USB
connector side) and solder them *on the flip side*.  It's easiest to do
them one at a time.

Now place the Teensy++ onto the ZIF socket, *remembering to leave the
bottom two pins exposed*, and start carefully soldering. 

Jumper wires:

AREF connects to PA1 just below the chip.
GND connects to PA2 just below the chip.

Facing the assembly, USB connector/latch up:

PA3 connects to the exposed pin to the right at the bottom of the ZIF
socket.

PA7 connects to the exposed pin to the left at the bottom of the ZIF
socket.

Now plug the board in and make sure the blinky runs and it enumerates.
Great.  Software is next.

# Putting Together the Toolchain

In addition to either this repository or the original, you'll need the
following:

* avr-gcc and libs
* avrdude (on some platforms, all the avr stuff is in one package)
* https://github.com/PaulStoffregen/teensy_loader_cli
  (See the instructions at https://www.pjrc.com/teensy/loader_cli.html)
* serial terminal emulator of your choice (minicom works great on Linux)
* lrzsz -- Tools for zmodem/xmodem/ymode file transfer

Props to Paul Stoffregen, the Teensy++ is a well put together and
documented project, which is refreshing to say the least. Please, RTFM
the instructions in the above link, think it through, you'll be fine.
Re-documenting that process is beyond the scope of this document.

NB: Bear in  mind that PROMdate's Makefile expects to find
teensy_loader_cli in ~/opt/teensy_loader_cli.  Tweak the Makefile
accordingly if you build in a different location.

# Date That PROM

To burn prom.hex to Teensy++:

	teensy_loader_cli -v --mcu=at90usb1286 prom.hex

On Linux this will cause the board to enumerate as /dev/ttyACM0

Use a separate 5V source for the PROM, make certain you are attached to
the correct pins.  Nice trick: a test lead the same diameter as the PROM
pin; the ZIF socket will clamp both pin and lead this way.

I used what a had: a PL2303 plugged into an adjacent USB port with the
5V and GND pins run out to leads and to the appropriate pins on the
PROM.

To "talk" to the PROM via the Teensy++:

Using minicom:

	minicom -D /dev/ttyACM0 

NB: If you can't connect, make sure minicom is set up properly:
115200 baud, 8N1.  'man minicom'

At prompt, type ? or help for a list of commands.  You should run 'l'
first.  Set your PROM type with m<PROM> from the list.  If you need to
extend the list of PROM's, you may do so in chips.c, but you will need
to re-make.

Next, check that you are able to read data off the PROM:

    >r0
    00000000 00 20 61 74 72 53 67 31 20 61 74 72 53 67 32 20  . atrSg1 atrSg2 
    00000010 61 74 72 53 67 33 20 61 74 72 53 67 34 20 61 74  atrSg3 atrSg4 at
    00000020 72 53 67 35 20 61 74 72 53 67 36 20 61 74 72 53  rSg5 atrSg6 atrS
    00000030 67 37 20 61 74 72 53 67 38 20 61 74 72 53 67 39  g7 atrSg8 atrSg9

You can read as much as you want manually, but it's handier to just try
to dump the data off the prom to a file at this point:

    > CTRL-A R  (R is for RECEIVE)

A pop-up window will appear.  Select xmodem.
A pop-up window will appear asking you to name the file.  Name it,
giving it the .bin extension and hit <ENTER>.

The board will flash like crazy and dump the PROM. The pop up window
will display a message like this:


         +----------[xmodem download - Press CTRL-C to quit]-----------+
         |rx: ready to receive TMS2764JL-45.bin                        |
         |Retry 0: Got 025 sector header                               |
         |Bytes received:    8192   BPS:1676                           |
         |                                                             |
         |Transfer complete                                            |
         |                                                             |
         | READY: press any key to continue...                         |
         +-------------------------------------------------------------+

Congrats, you have now dated a PROM.

To use hex2png:

    hex2png <filename.bin> > <newname.png>

To view newname.png:

    display newname.png (display requires ImageMagick).

# Coda

This is an easily extensible project: more devices can be added to
chips.c, and you can tweak things to your hearts desire.  All that is
required is rebuilding with make after making your changes.

Major props to Trammell Hudson et. al and Paul Stoffregen. If you have
to work with old hardware (me -- recovering data from industrial
equipment -- dying CNC's for example --  and other devices) this is a handy 
bit of kit to have in your toolbox.  

It could be a fun project for kids or a class as well, as anything that
has the remotest whiff of Dark Magic has an alarming allure for the
young.



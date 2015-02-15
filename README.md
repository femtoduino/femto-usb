femto-usb
=========

FemtoUSB - Atmel ARM Cortex M0+ (SAMD21), 256KB @ 48MHz, 3V3

This micro controller is designed as a basic starting point for those interested in ARM designs (especially if you are transitioning from 8-bit AVR chips).

We use Atmel's ARM Cortex M0+ offering, the ATSAMD21E18A. The schematic follows the suggested design found in the SAM D21 data sheet.


## Why Atmel?
Atmel has some of the best support for the open-source hardwaree community. They offer proper documentation, excellent chip performance, and a great foundation via the Atmel Software Framework.
Did we mention how easy it is to get started with ARM using ATMEL?

## Key similarities, and differences between 8-bit AVR chips, and newer ARM chips

So, you've been working with Arduino boards for a while now, but want to get into ARM chips right? How difficult can it be?

As it turns out, it's much less so today than it was a short time ago. Thankfully, most micro controllers have a similar set of requirements:

 1) Add in some snubbing caps, as per the chip's data sheet.
 2) Add in some resistors, as per the chip's data sheet.
 3) Add the reset circuit.
 4) Add a "power on" LED, making sure to wire up voltage and ground pins correctly.
 5) Hook up a USB port, as per the chip's data sheet.
 6) Burn a boot loader.

 ...You can of course, add in a reverse current protection circuit, a crystal clock source for chips that don't have an internal one (or if you want a faster clock source), some fancy peripheral additions, etc... but at the most basic design, both 8-bit AVR chips and ARM chips have similar design requirements.
 
 However, here's where we will draw an imaginary line, and now distinguish between 8-bit AVRs and 16/32-bit ARM chips.
 
 When working with ARM chips, you will need a programmer dongle to initially burn a boot loader. 
 "Serial Wire Debug" seems to be the most basic form of the (JTAG)[http://en.wikipedia.org/wiki/Joint_Test_Action_Group] interface - something provided by all ARM chips. 
 This is akin to the "Ardiuno ISP" mode of programming. 
 Keep in mind, however, not all chips "talk the same" between the chip and a dongle. Kind of like how two people can have the same interface (vocal chords), but speak different languages. 
 Fortunately, it seems Atmel's SAM D21 chips talk "CMSIS" (Cortex Microcontroller Software Interface Standard)[http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php], which is a vendor-independent hardware abstraction layer for the Cortex-M processor series.
 This is another great reason to use Atmel's line of ARM chips, for what it's worth.
 
 The pins used to provide a JTAG connection vary depending on the ARM chip used, and provide more debugging features when more pins are added.
 For the most part, JTAG "Serial Wire Debug" establishes these five (5) connections:
 
 * Ground
 * Voltage Reference (Does not supply voltage, you must tie your regulated voltage source to this line when connecting a dongle to a board)
 * RESET
 * SWCLK (Serial Wire Clock, sometimes overlaid on the "TCK" pin)
 * SWDIO (Serial Wire Debug Input/Output, sometimes overlaid on the "TMS" pin)
 
Your programming dongle should have a data sheet telling you the pin out provided, so you can wire it to your board accordingly.

Another key difference, *and a very important one at that*, is the voltage!

You may be used to 5V logic levels working with Atmel's 8-bit chips... but 5V can be utterly destructive to an ARM chip, as they are meant to work with less power.
The usual voltage range for Atmel's line of ARM chips is somewhere between 1.8V to 3.3V. 
Failure to heed this notice will likely result in the production of a tiny paperweight, instead of a micro controller. 

Ok, I suppose we can move onward. Let's get everything we need to build one.

## Building your own board
The following instructions should help you build your own ARM board. Please follow common workplace and ESD safety practices, and ensure proper ventilation. Solder paste, solder rework, PCB fibers, hot air rework nozzles, soldering irons, and so on, should all be handled with proper safety in mind.
Please observe you local materials safety and disposal laws/guides. We assume no liability whatsoever. Not even. Just no.

### Equipment
As with all ARM chips, you will benefit greatly from having a programmer dongle.
Some vendors lock up their chips behind really expensive software tools, and even more expensive programmer dongles ($200+). 
Plus, their support is lacking, and assume you are some kind of command line compiling ELF/Hex guru. 
It's great if you are one already, but we all have to start somewhere, right?

Thankfully, Atmel offers their ATMEL-ICE programmer at a reasonable price (about $85 USD)[http://store.atmel.com/PartDetail.aspx?q=p:10500375#tc:description]
(I hear you can get them much cheaper without the case, though don't expect it to come with ribbon cables if you go the cheap route.)

Please source the following equipment:
* An ATMEL-ICE programmer. See http://www.atmel.com/tools/atatmel-ice.aspx
* A Windows, Mac OS X, or Linux machine. (Though you may initially require a Windows machine to burn the SAM-BA bootloader using Atmel Studio.)
* Two (2) USB "Micro B" cable. Some cables are less than useless. Try to use ones without a choke if you get any trouble with USB connections.
* Round jumper wires.
* Bread board.
* A reflow station. We use a Sparkfun Hot-air Rework Station: https://www.sparkfun.com/products/10706 
* A good pair of tweezers. See Adafruit's offering: http://www.adafruit.com/products/421?gclid=CN-E8eah5MMCFU9efgodj3IAJQ
* Helping third hand. http://www.adafruit.com/product/291
* A multimeter.

Additionally, you will need the following supplies:
* Low temp Lead-free solder paste. We recommend Pieco's  paste press dispenser bundle with the ChipQuik T5 paste. https://www.tindie.com/products/Pieco/paste-press/  
* Some PCBs. You can order them from OSHpark https://oshpark.com/shared_projects/F0wPGZvV, or open up the Eagle PCB designs to modify and make your own (see the FemtoUSB_r1.0.0 folder, open up the femtoUSB_r1.0.0_TQFP board and schematic)
* The SMD components. Thanks Octopart! https://octopart.com/bom-lookup/UgbhotEt ...the gerbers and parts lists can also be found within the `FemtoUSB_r1.0.0/femtoUSB_r1.0.0_TQFP/` folder

### Software tools
On windows, you will need
* Atmel Studio (6.2.x as of the time of this writing)

On Mac OS X, and Linux
 * Terry Guo's GNU ARM Embedded Toolchain. https://launchpad.net/~terry.guo/+archive/ubuntu/gcc-arm-embedded (Ubuntu user's can add a PPA and have the apt-get package manager install it)
 * Legit command line skills.
 * Make tools.
 
 For any machine you are on, you will also need the following:
 
 * Atmel Software Framework (ASF) http://www.atmel.com/tools/avrsoftwareframework.aspx?tab=overview
 * Atmel SAM-BA In-system Programmer http://www.atmel.com/tools/ATMELSAM-BAIN-SYSTEMPROGRAMMER.aspx
 * "Atmel AT07175: SAM-BA Bootloader for SAM D21" http://www.atmel.com/devices/atsamd21e18a.aspx?tab=documents ...Note, this is what actually gets programmed on to the chip so we can load stuff via USB instead.
 
 
### Assembling a board, or three.
For the most part, try not to make a mess of solder paste everywhere. Add just enough to cover the solder pads with a thin layer, and place the correct part according to the femtoUSB_r1.0.0_TQFP_parts.csv spread sheet.
If you don't have the 4-layer or more version of Eagle PCB, you can also look at the femtoUSB_r1.0.0_TQFP.pdf file for schematics and board drawings. See `FemtoUSB_r1.0.0/femtoUSB_r1.0.0_TQFP/`

Once you have everything assembled, make sure your board is secured by the "helping third hand" tool, and remove any nozzle attachments from your reflow station to get the widest air flow. 
Heat up your reflow station to around 200c - 210c or so. The low temperature lead-free solder paste should not require too much heat. Air flow should be kept low (I dial it down to level 4 on my Sparkfun Hot-air Rework station).
Go ahead and adjust the temperature and air flow as you see fit. A constant stream of hot air should reflow stuff within a minute or less.

Be certain to keep the air flow on the USB connection a bit longer than the rest of the board, as the pads underneath the USB connector may take a few seconds longer to heat up.

*Allow your boards to cool down on their own before handling!*

### Testing and programming
Grab you multimeter, switch it to the "open circuit" checking mode (you know, the one that beeps when touch the two leads).
Test between the GND (Ground) pin and all other pins to assert there are no shorts between Ground and the rest of the pins. 
Do the same between voltage and other pins. As a last step, you can also test by steping pegs, (testing between two concurrent pins, moving up by one pin at a time).
This last step is mostly necessary if you have a hard time checking the board for shorts or gaps visually.

Now, it's time to give our board(s) a bootloader!


Wire the following pins together, tying the two "GND" pins from the ATMEL-ICE programmer dongle on to a breadboard, so you can use a single wire from the breadboard GND line to the board's GND pin.
Additionally, you will need to make sure the board's 3V3 (regulated) volatage pin is tied to the programmer dongle's VTG (voltage reference) pin, as the programmer dongle does *not* supply voltage. It merely detects the voltage level with the VREF line.

*DO NOT MIX UP THE "SAM" PORT WITH THE "AVR" PORT, AS THE PINOUT IS DIFFERENT!*

| ATMEL-ICE "SAM" port           | Your Board     |
|--------------------------------|----------------|
| SWDCLK (SAM Pin 4)             | SWDCLK (PA31)  |
| SWDIO (SAM Pin 2)              | SWDIO (PA30)   |
| nRST (SAM Pin 10, labeled "0") | RESET          |
| VTG (SAM Pin 1)                | 3V3 (Regulated)|
| GND (SAM Pin 3, 5)             | GND            |


Open up Atmel Studio on your Windows machine, plug in the USB cable for your board, plug in the USB cable for your programmer dongle.
Your board should power on, the programmer should have both a red LED and a green LED powered on.

If only the red programmer dongle LED turns on, check for loose connections and check for any solder issues on your board. At worse, build another one with a different PCB and try again.
Don't worry about the semi-on blue LED on your board. That's just the SWDCLK line. You can unsolder it if it bugs you. 

Once you have a good board hooked up, and Atmel Studio running, open up the Device Programmer dialog by navigating to the Tools -> Device Programmer menu item.

In the Device Programmer dialog, select "ATMEL-ICE" as the tool, select "ATSAMD21E18A" as the device (double check to be certain you read this right), and select "SWD" as the interface. 
Click the "Apply" button. Set the "SWD Clock" value to anything from 32KHz to 2MHz and click "Set".
2MHz is plenty fast, but if you ever run into issues reading the chip, try going as low as 32KHz.

You should now click the "Read" button under "Device Signature" to see if you get back a hex value. The board's blue LED will go from semi-off, to bright blue.
If you get a message saying your device is unreadable, double check your connections, double check your solder work, and try again.

Assuming you got back a hex value under "Device Signature", congrats! You appear to have a functional board!

Let's now burn the bootloader.

Click the "Memories" section of the Device Programmer dialog. In the "Flash" area, use the file navigator button to browse to the downloaded (and extracted) `Atmel-42366-SAM-BA-Bootloader-for-SAM-D21_ApplicationNote_AT07175/load sam-ba/samd21e18a/` folder, and select `samd21_sam_ba_usbcdc.hex`
Assert the "Erase Flash before programming" and "Verify Flash after programming" checkboxes are checked. Click "Program". After a brief moment, the Device Programmer dialog should come back with:

```
Erasing device... OK
Programming Flash...OK
Verifying Flash...OK
```


Success! You now have a board with the SAM-BA bootloader! If you're on a Windows machine, you may notice Windows will now attempt to install drivers for the new device. The Windows driver install tool should succeed in finding the correct drivers.
You can now see your board on Linux/Mac as well. Simply plug it in via USB, and run the 'lsusb' command from a terminal. You will see an Atmel 'SAM-BA' device listing.

## Next Steps
We are working on getting Arduino integration working, along with other non-Arduino tools to load stuff via USB. Stay tuned!


# Getting started with the esp8266 12F board
![](https://i.imgsafe.org/185696f7bf.jpg)


## Wiring up to a breadboard
Required..

1.ESP8266 12F
1.USB to UART board. e.g SiLabs CP2102
1.Power adapter (12v 1a)
1.Power supply adapter (To power the breadboard) e.g elego power mb v2. MUST SUPPLY 3.3V NOT 5V
1.3 10k resistors (used as pull-ups)
1.Push button
1.ds18b20 (optional) w/ 4.7k pullup resistor (optional) 
1.Lots of breadboard wires
1.Ardunio ide (configured for your USB - UART board) 

The wiring of this module can very tricky. Firstly, the pitch of the holes means that they do not line up with a standard breadboard. Secondly, they are very small.
For testing, I found that bending some male breadboard wires around the contacts worked fine.

In a basic setup, wire up as follows. (temp sensor is optional)..
![](https://i.imgsafe.org/196523f85e.png)

The important PIN is GPIO 0 - This determines the boot mode (e.g Boot program from ROM, boot into flashable mode)

1. VCC - 3.3v Rail
1. GND - Ground rail
1. GPIO 15 - Ground (via 10k resistor)
1. GPIO 0 - Ground (via 10k resistor)  -- FLASHING MODE
1. GPIO 0 - Floating (Not connected) -- NORMAL BOOT MODE
1. GPIO 14 - to ds18b20 data pin (middle pin) - Optional
1. EN - 3.3v Rail (via 10k resistor)
1. TX - RX of your UART controller
1. RX - TX of your UART controller

USB-UART Controller
1. TX > RX of ESP8266 
1. RX > TX of ESP8266
1. GND > GND Rail

**(DO NOT POWER THE ESP8266 FROM THE USB-UART)**

All other pins can be left floating.

Heres how it looks on the breadboard 
![](https://i.imgsafe.org/19bf0ccadc.jpg)

![](https://i.imgsafe.org/19c682a907.jpg)

## Viewing the Serial output..
(Its worth reading the notes and troubleshooting below before getting to this step.)

Most esp8266's come with a preloaded firmware called AT. This is a very basic interface to the chip, you can of course flash it with whatever you like.
1. To confirm your wiring is correct, attach the 3.3v supply to the breadboards vcc/gnd rails.
1. Attach 3 cables from your USB-UART connector as instructed above. 
1. Open up the ardunio ide and open the serial monitor. Set the baud rate to 74880 and the com port to your USB-UART device.
1. Reset the esp8266 by disconnecting and reconnecting the power.
1. You should see something like the following in the serial output...

>   
    ets Jan  8 2013,rst cause:2, boot mode:(3,7)

    load 0x4010f000, len 1384, room 16 
    tail 8
    chksum 0x2d
    csum 0x2d
    v09f0c112
    ~ld
    -- Lots of gibberish here.`
This is actually a good sign. It means the boot loader has finished and it has loaded the installed program, the baud rate has been changed to 115200 hence the reason you cannot read the output.
Simply change the serial monitors baud rate to 115200 and reboot the ESP, power off/on.

Now when it boots, you should see firstly (some gibberish, that was the above) followed by "Ready" if you have the AT firmware installed, or, the output of whatever program is installed.
- You can have a play with the AT commands by referencing [this](https://www.itead.cc/wiki/ESP8266_Serial_WIFI_Module)

## Writing to the board (firmware)
The easiest way to do this is using the ardunio ide.
* Add the board library to the ide. ([Using this guide](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/installing-the-esp8266-arduino-addon))
* Make sure the board is wired as per wiring diagram.
* Set serial monitor baud rate to 74880 (so we can view the boot loader mode).
* Hold down the push button, power on.

If you see the following...  

>   ets Jan  8 2013,rst cause:2, boot mode:(1,7)

Then congrats, the device is in flash mode and is waiting for that shiny new firmware. You can now let go of the button. (The ESP will remain in this flash mode until it reboots)

You can now either flash your own .ino, or use the example below. This example will turn the ESP into a wifi point, allow you to connect to it and view the temperature readout from the 1 wire sensor on the breadboard.

In order to push this new firmware we need to make sure your Ardunio setup has firstly, the ESP support files and secondly you need to add the OneWire library..

>.   Sketch > Include Library > Manage Libraries > Search for OneWire and select the "OneWire" library.

The board should already be installed as above, but you can check by heading into 
>.   Sketch > Include Library > "Contributed Libraries " > There should be a number of ESP... libraries.

Copy the sketch into a new .ino project.
>
    You can find the .ino file here.... 

Change the two WIFI variables to suit your access point.
Make sure that the ardunio board is set to "Generic ESP8266 Module".
Confirm that your settings are as follows (Your port may be different..)

![](https://i.imgsafe.org/1936fac932.png)

Click the upload button and watch the status bar, you should see .......................... [31%] etc...
Once the upload has completed, your sketch may already be running on the ESP but for best practise, turn it off and on again. (Since we are not holding the button down, it will load in normal mode).
Watch the serial monitor in Ardunio (make sure its set to 115200).
You should see..

>
    Connected to [YOURACCESSPOINT]
    IP address: [THE ESP'S IP address]
    MDNS responder started
    HTTP server started

(If you didn't see that, then double check your AP details in the .ino sketch and reupload. - Turn off, hold button, power on, release button).
Type in the IP address to your browser... eg.. http://192.168.1.1.
You should see..
>
    hello from esp8266!

Next, type in /gettemp.. So the url should look like http://espipaddress/gettemp and hit Go.
You should see both the page has returned the sensor value...

> 
    The Temperature is 21.12 / 70.03

and in the serial monitor...

>
    ROM = 28 FF 5C E1 74 16 4 6C
    Chip = DS18B20
    Data = 1 52 1 4B 46 7F FF C 10 6E  CRC=6E
    Temperature = 21.12 Celsius, 70.03 Fahrenheit

If you see these, Success!! You can remove the USB-UART connector now and run the module on its own.
   
(If you don't see. value, confirm that the temp sensors data pin is connected to gpio14 and that you has the 4.7K resistor off the data pin to 3.3v).

>    The one wire sensor can run off 3.3v just fine in most cases, but a 5v supply is probably preferred (although you would have to be very careful running such voltages through a sensor on the ESP8266 as its not 5V tollerant )

- That concludes the intro into this chip. The next steps will most likely be..

>
    Build a power supply circuit to regulate 3.3v (Maybe use something like the AMS1117 regulator)
    Upload the results to an API / Database at an interval.
    Run the ESP off a battery, making it completely wireless.

## Notes..
When the board first starts up, it enters into the boot loader. This boot loader firstly tells you which mode the chip has booted up into (including any resets) but it also gives some useful diagnostic info.
In order to view this in the serial monitor, you must switch baud rate to 74880 and power on. Once the boot loader  has finished, provided the pins dictate so, it loads the firmware stored in the rom. It will usually switch to 115200 (If the stock AT firmware is installed). So again, you need to switch the baud rate to view the actual program serial output.

## Troubleshooting.

### Bad PSU
* Device doesn't fully boot, scrambled message etc.
Check your PSU, this is the most common route of all evil regarding these chips. They require a fairly high burst of mA when starting up. Its recommended to use something like a 12v 1amp transformer into a dedicated psu for testing on a breadboard. (The PSU should have all the necessary 3.3v regulation, capacitance over the vcc/gnd lines).
When using a cheap usb type charger (5v 1a) although the thing powered on, it would not get past the boot loader. 
You may see just the following ... 

`ets Jan  8 2013,rst cause:2, boot mode:(3,7)`

Of-course, this can caused by bad PIN setup, but in testing, an underpowered psu gives also gives this boot mode (When it should be firing up from the Rom) - Even if your setup is all correct.

Alternitavly, it may actually get further than that, but then you see...

`ets Jan 8 2013,rst cause:2, boot mode:(3,7)`

`load 0x40100000, len 25460, room 16`
`tail 4`
`chksum 0x60`
`load 0x3ffe8000, len 1360, room 4`
`tail 12`
`chksum 0x53`
`ho 0 tail 12 room 4`
`load 0x3ffe8550, len 10428, room 12`
`tail 0`
`chksum 0xb3`
`csum 0xb3`
`csum err     `
`ets_main.c`

The good news is that the mere fact that the boot loader is running _some_ output means that it is probabaly OK, so the error isn't too much of a worry (in most cases).
Again, the first thing to check is your psu as the above error was noted from an underpowered psu (5v 1a).

## CP2102 USB to UART (When using MAC SIERRA)
You will need to manually install [the Mac drivers on this page](http://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

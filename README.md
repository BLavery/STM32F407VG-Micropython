# STM32F407VG-Micropython
Notes on putting micropython on STM32F407VG $10 bare board<img align="right" src="images/ss5.png">

A pyboard on steroids? Well, it has more GPIO pins, anyway.

For the moment, cross-referencing with the following site may be of help
in understanding this board:
https://github.com/BLavery/STM32F407VG-Arduino

The DIY board MCU is '407VG. This is the same MCU as on the STM32 F4 Discovery board, which has a micropython firmware build.

We can load that micropython binary on our DIY board. Here is what I did:

 - In boot mode (reset with jumper on boot1),
 - using linux (debian mint),
 - which has dfu-util available,
 - with merely a USB connection,
 - using the standard latest firmware image (STM32F4 Discovery) from here: http://micropython.org/download 
 
... we can put the PYBOARD micropython image on this board:

 - sudo dfu-util -a 0 -D pybv10-20181205-v1.9.4-712-gc6365ffb9.dfu      (our dfu file)
 
THAT'S SO EASY.

Swap the boot jumper from boot1 to boot0 and press reset to re-enable run mode. 

Here we have a slight glitch. The "real" F4 DISC board has a voltage sensor on PA9 to detect that the USB is plugged in. Our DIY board doesn't have that, but the code we installed thinks we have a real F4 DISC. We can fudge a HIGH  voltage on PA9 if we can get these 2 lines in file main.py on the python. But we can't get at that file until the USB starts.

Solution: temporarily connect a jumper wire from +3V to PA9. Reset.  A new "drive" appears on the filemanager, "PYBFLASH", with empty boot.py and main.py already present.  Use any editor to add these 2 lines to main.py:
```
from machine import Pin
Pin("PA9", Pin.OUT).high()
```
This will hold PA9 high. Save. Unmount the PYBFLASH drive. Remove the temporary jumper. Reset. The PYBFLASH drive should return. Glitch finished.

USB now gives us a serial connection as well. Run a serial terminal utility. (I use gtkterm). On linux my port name is /dev/ttyACM0 or similar. I'm sure you Win or Mac folk can work out your equivalent.

A crtl-C and ctrl-D get you in and out of the REPL prompt of micropython. 

Our DIY board has a LED on PE0. Use any editor to upgrade main.py to this:

```
from time import sleep
from machine import Pin
Pin("PA9", Pin.OUT).high()
led=Pin("PE0", Pin.OUT)
while 1:
  led.value(1-led.value())
  sleep(1)

```

Save. Reset board. Lo, the led blinks. 

A new world beckons.

---



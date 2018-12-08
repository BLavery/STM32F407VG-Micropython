# STM32F407VG & Micropython
Notes on putting micropython on __STM32F407VG $10 bare board__<img align="right" src="images/ss5.png">

A pyboard on steroids? Well, it has more GPIO pins, anyway.

For the moment, cross-referencing with the following site may be of help
in understanding this board:
https://github.com/BLavery/STM32F407VG-Arduino
and especially on how to set boot jumper and reset into dfu (bootloader) mode.

The DIY board MCU is '407VG. This is the same MCU as on the STM32 F4 Discovery board, which has a micropython firmware build.

We can load that micropython binary on our DIY board. Here is what I did:

 - In boot mode (reset with jumper on boot1),
 - using linux (debian mint),
 - which has dfu-util available,
 - with merely a USB connection,
 - using the standard latest firmware image (STM32F4 Discovery) from here: http://micropython.org/download 
 
... we can put the PYBOARD micropython image on this board (bootloader mode, jumper on boot1, press reset):
```
sudo dfu-util -a 0 -D STM32F4DISC-20181207-v1.9.4-725-gd690c2e14.dfu      (our dfu file)
``` 
THAT'S SO EASY.

Swap the boot jumper from boot1 to boot0 and press reset to re-enable run mode. 

Here we have a slight glitch. The "real" F4 DISC board has a voltage sensor on PA9 to detect that the USB is plugged in. Our DIY board doesn't have that, but the code we installed thinks we have a real F4 DISC, and USB won't start. We can fudge a HIGH  voltage on PA9 if we can get a fix into file main.py on the python. But we can't get at that file until the USB starts.

Solution: temporarily connect a jumper wire from +3V to PA9. Reset.  A new "drive" appears on the filemanager, "PYBFLASH", with empty boot.py and main.py already present.  Use any editor to add these 2 lines to main.py:
```
from machine import Pin
Pin("PA9", Pin.OUT).high()
```
This will hold PA9 high. Save. Unmount the PYBFLASH drive. Remove the temporary jumper. Reset. The PYBFLASH drive should return, without needing the jumper wire. Glitch finished. Remember not to use PA9 for other things in future.

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

http://docs.micropython.org  (.. and follow the pyboard specifics, not esp8266 or wipy. Because pyboard also uses STM32F407!)

---

So we now have the STM32F4 Discovery board micropython running on our DIY board.
There are a few hardware differences (read: things NOT fitted on our DIY board) that would be good to have some
firmware adjustments for:
 - No sensing of "USB plugged in" voltage (We had a workaround above)
 - Not 4 LEDs, just one, on a different pin. Active low, not high.
 - The USR button on different pin. Active low, not high.
 - The pins for UART1 no longer conflicted by the "USB plugged in" sensing
 - No SD flash slot fitted to board
 
It's not too difficult to recompile the micropython firmware to exactly match our DIY board.
I did it on my linux PC, set up as per the file compile-micropython.doc. 
I added the STM32F407 folder from here into my .../micropython/ports/stm32/boards/ folder,
alongside the other board variants. The new variant board is just a clone of STM32F4DISC, with config tweaks to only 2 files.

Positioning my terminal into .../micropython/ports/stm32 folder:
```
    make STM32F407
```
In a few minutes I have fresh new firmware called "firmware.dfu" in a newly-created folder .../micropython/ports/stm32/build-STM32F407. To upload, change directory to that, and, as before, change boot jumper, reset to bootloader, and
```
   sudo dfu-util -a 0 -D firmware.dfu
```

But you shouldn't need yourself to recompile. I have put a compiled firmware here for your download. Still faithfully just called firmware.dfu.
It should work cleanly. The USB starts automatically now, without the glitch-fixer above. Even the Safe Mode and Flash Volume Reset work. (You get single/double/triple LED flashes, not 3 colours.)  See http://docs.micropython.org/en/latest/pyboard/tutorial/reset.html

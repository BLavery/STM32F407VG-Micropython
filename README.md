# STM32F407VG-Micropython
Notes on putting micropython on STM32F407VG $10 bare board<img align="right" src="images/ss5.png">

A pyboard on steroids? Well, it has more GPIO pins, anyway.

For the moment, cross-referencing with the following site may be of help
in understanding this board:
https://github.com/BLavery/STM32F407VG-Arduino

The DIY board MCU is '407VG. This is the same MCU as on the STM32 F4 Discovery board, which has a pyboard derivative micropython build.

We can steal its micropython binary to use on our DIY board. Here is what I did:

 - In boot mode (reset with jumper on boot1),
 - using linux (debian mint),
 - which has dfu-util available,
 - with merely a USB connection,
 - using the standard latest firmware image (STM32F4 DISC) from here: http://micropython.org/download 
 
... we can put the PYBOARD micropython image on this board:

 - sudo dfu-util -a 0 -D pybv10-20181205-v1.9.4-712-gc6365ffb9.dfu      (your dfu file)
 
THAT'S SO EASY.

Swap the boot jumper from boot1 to boot0 and press reset to re-enable run mode. A new "drive" appears on my filemanager, "PYBFLASH", with empty boot.py and main.py already present.  

Our DIY board has a LED on PE0. Use any editor to put this into main.py:

```
import machine, time
time.sleep(3)
PE0 = machine.Pin.cpu.E0
led=machine.Pin(PE0, machine.Pin.OUT)
while 1:
  led.high()
  time.sleep(1)
  led.low()
  time.sleep(1)
```

Save. Reset board. Lo, a led blinks. 

To get serial communication, we still use the USB connection. Run a serial terminal utility. (I use gtkterm). On linux my port name is /dev/ttyACM0 or similar. I'm sure you Win or Mac folk can work out your equivalent.

A crtl-C and ctrl-D get you in and out of the REPL prompt of micropython. 

A new world beckons.


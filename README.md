# STM32F407VG-Micropython
Notes on putting micropython on STM32F407VG $10 bare board<img align="right" src="images/ss5.png">

A pyboard on steroids? Well, it has more GPIO pins, anyway.

For the moment, cross-referencing with this site may be of use in understanding this board:
https://github.com/BLavery/STM32F407VG-Arduino

The DIY board MCU is '407VG. This is similar enough to '405RG used on the PyBoard. http://docs.micropython.org/en/v1.9.2/pyboard/pyboard/general.html

We can steal its micropython binary to use on our DIY board. Here is what I did:

 - In boot mode (reset with jumper on boot1),
 - using linux (debian mint),
 - which has dfu-util available,
 - with merely a USB connection,
 - using the standard latest pyboard v1 firmware image from here: http://micropython.org/download 
 
... we can put a PYBOARD micropython image on this board:

 - sudo dfu-util -a 0 -D pybv10-20181205-v1.9.4-712-gc6365ffb9.dfu      (your dfu file)
 
THAT'S SO EASY.

Swap the boot jumper from boot1 to boot0 and press reset to re-enable run mode. A new "drive" appears on my filemanager, "PYBFLASH", with empty boot.py and main.py already present.  Our DIY board has a LED on PE0. Use any editor to put this into main.py:

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

To get serial communication, attach a 3V uart tp TX/RX pins (PA9 / PA10) and run a serial terminal utility. (I use gtkterm). A couple of presses, and the REPL prompt of micropython is there. 

A new world beckons.


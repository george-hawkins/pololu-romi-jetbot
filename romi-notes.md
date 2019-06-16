Romi notes
==========

References:

* [Pololu guide](https://www.pololu.com/blog/663/building-a-raspberry-pi-robot-with-the-romi-chassis) to building a Raspberry Pi robot with the Romi chassis.
* Romi 32U4 control board [user guide](https://www.pololu.com/docs/0J69/all).

The mounting holes either end of the GPIO header on the Jetson Nano match those of the Raspberry Pi. However the other pair of mounting holes are further away than on a Raspberry Pi, so these cannot be attached to the Romi control board as one could with a Pi. The Jetson is substantially larger than the Pi - so, unlike the Pi, it cannot be mounted upside down with its GPIO male header plugged into the corresponding female header on the control board. The Nano's size also means far more of the control board is inaccessible if its mounted directly onto it.

We're going to go through the build guide but use a Jetson Nano rather than a Pi. The build guide mentions an optional LCD character display - the Nano will cover the area where this display can be connected (and anyway its easier to report information via the Nano).

32U4 control board
------------------

The most important componet is the 32U4 control board - read through its [user guide](https://www.pololu.com/docs/0J69/all). Despite being quite long, its important to understand how it works and what its capable of.

The control board has its own MCU that will act as a slave to the Nano, taking care of controlling the motors without the Nano having to worry about the mechanics of what's going it - it just has to issue high level commands.

The control board user guide mentions using a charger like the [iMAX-B6AC V2](https://www.pololu.com/product/2588) to recharge the NiMH AA batteries that can be used with the Romi. A charger like this is the right kind of thing for balance charging LiPos and the like - and while the idea of using a single charger for everything sounds nice, I've come to the conclusion (from experience and a lot of googling) that you're better off using a dedicated NiMH AA charger like the [8-cell Panasonic BQ-CC63](https://www.panasonic.com/global/consumer/battery/eneloop/chargerlineup.html).

User LEDs:

* The yellow LED is connected to Arduino digital pin 13 and lights when the pin is driven *high*.
* The red LED is connected to Arduino digital pin 17 and lights when the pin is driven *low*.
* The green LED is connected to Arduino digital pin 30 and lights when the pin is driven *low*.

There's also a non-user blue LED (on when battery powered) and green LED (on when USB powered).

There are three user pushbuttons, a power button (rear left) and a reset button (front right).

Note that the LEDs, buttons and LCD all share pins - but this is largely transparent to you if you work with them via the Romi Arduino library.

There's a tiny little potentiometer above the pins for the LCD (it's meant for controlling the LCDs contrast but if you're not using the LCD then you can use it for other purposes).

Controling the motors involves four pins - one each for speed and direction for left and right.

There's a voltage divider that allows you to monitor the voltage decreasing as the batteries run out using `readBatteryMillivolts()`.

Four pins are used to read the data from the quadrature encoders - the encoders provide a feedback loop, you can tell the motors to turn faster but how much the wheels actually turn is dependent on various factors (they may not turn at all if the robot is stuck) - the encoders report the actual turning.

The board has an LSM6DS33 - a 6-DOF accelerometer and gyro. It's connected to the boards I2C bus. Pololu suggest reading the LSM6DS33 datasheet but I'm not sure how much is gained by that - their product page for their separate [LSM6DS33 breakout](https://www.pololu.com/product/2736) probably contains as much information as most people would want (note that on the Romi control board only I2C is supported whearas the breakout also supports SPI).

The LSM6DS33 has the I2C slave address 1101011, i.e. 0x6B. This address is also used by the 9-DOF LSM9DS0 (see the Adafruit [table of I2C addresses](https://learn.adafruit.com/i2c-addresses/the-list)) which also features a magnotometer. A magnotometer would have been a nice addition for dead-reakoning but perhaps it would have been useless on the control board (as a result of being overwhelmed by electrical noise from the motors).

Note: one could use e.g. an [LSM303D](https://www.pololu.com/product/2127) as a separate compass and raise it up on a mast to escape electrical noise (as is common on drones where this is generally done with a combined GPS and compass). There are no end of options when it comes to compasses combined with other sensors, so this would require more investigation for a given setup - the LSM303D is just mentioned here as an _example_.

The slider switch should *always* be left in the off position - otherwise it disrupts the proper functioning of the push button power switch. The slider only comes into proper use if you cut the _Btn Jmp_ jumper and use it instead of the push button power switch. This looks like a potential source of confusion!

TODO: it's possible to get "your robot to turn off its own power" - see if this is covered in the Raspberry Pi guide. If not see about adding this as something triggered at shutdown via systemd. See the last part of the "Power switch circuit" section.

The power regulator has a maximum continuous output current of a little under 2.5A at 5V when the input voltage is 7.2V (as it would be from 6 NiMH AA batteries). See the regulator [product page](https://www.pololu.com/product/2858) for more details, including graphs covering efficiency and current output.

TODO: try powering the Pi seperatly and see how much current the MCU, encoders, motor drivers and motors draw - the motors have a stall current of 1.25A each and apparently you should run such motors at around 20% to 30% of this (see [here](https://www.pololu.com/product/1520/faqs). So 25% of the two motors would be 1A which doesn't leave much for the Nano.

**Update:** according to this [post](https://forum.pololu.com/t/rpi5v-on-romi-control-board-maxes-out-at-500ma/17339/3?u=ghawkins), from Kevin at Pololu, "the motors are not powered from the regulated voltage, so the current they draw doesn’t reduce what’s available from VREG" - this seems to contradict the control board user guide that says "VREG is generally used to supply logic power for the ATmega32U4, **motor drivers**, and encoders. The rest of the regulator's achievable output current [...] can be used to power other devices [...]. Under typical conditions, up to 2 A of current is available from the VREG output."

2A is the maximum the Nano can draw via the USB power connector but it can apparently consume a lot more if powered via the the barrel jack - it'd be interesting to monitor the Nano's current draw, does it max out at 2A (implying it might consume more if it were available)?

For more on powering the Nano and why you should switch to 4A at 5V see this Jetson Hacks [page](https://www.jetsonhacks.com/2019/04/10/jetson-nano-use-more-power/).

The control board can safely be plugged into battery power and USB at the same time - it will prefer battery power over USB so to avoiding wasting power make sure battery power is swtiched off when working with the board via your computer.

Similarly it's safe to have the Pi connected to both its own power source, e.g. USB or barrel jack, and to the power meant for it from the control board (this safety is handled by the control board, i.e. it doesn't depend on Pi protection circuitry).

While the control board can power the Pi - the reverse isn't true, it can't draw power via the Pi.

The control board power to the Pi can be switched off by pulling the RPISHDN pin to 5V (it's the end pin of the block of four power related pins near the main header block for the Pi - see [here](https://a.pololu-files.com/picture/0J7535.1200.jpg)).

See the "power distribution" section for definitions for VBAT, VREG etc.

The ATmega32U4 of the control board operates at 5V, while the Pi operates at 3.3V - the control board talks to the Pi via level-shifted I2C.

In a minimal setup just four pins need to be connected from the board to the Pi - ground, RPI5V and the clock and data pins (SCL and SDA) of the I2C bus (which connects the Pi to the IMU as well as the MCU).

There are three level shifter blocks:

* LS1 allows the Pi to _read_ two 5V signals, i.e. have them converted down to 3.3V.
* LS2 and LS3 - these handle just one signal each - like LS1 are unidirectional but whether they convert from 5V to 3.3V or vice-versa can be configured.
 
For a good overview of how the MCU pins are assigned see the table in the "pin assignment" section.

Of the four AVR timers only timer3 is free - timer0 is used for `millis()`, timer1 for driving the motors and timer4 for the buzzer (there is no timer2).

For connecting additional devices, including further I2C devices, and for freeing up additional pins see the "adding electronics" section.

To manage servos you'll need to modify the standard servo library as described in the the "controlling a servo" section.

Soldering:

* You need to solder on the buzzer and the low profile headers for the encoders (we won't use the header for the LCD).
* The battery connectors need to be soldered on in place, i.e. you need to insert them into the chassis, attach the control board and then solder the connectors to the board. You can still remove the board (with connectors) afterwards.

---

The chassis should be assembled as described in the separate Romi chassis [user guide](https://www.pololu.com/docs/0J68/all).

---

Download the [Arduino desktop IDE](https://www.arduino.cc/en/Main/Software) and install it as described [here](https://www.arduino.cc/en/Guide/HomePage).

For Linux you just need to download it, unpack it and then run the contained `arduino` script.

Install the board description for the "Pololu A-Star 32U4" as described in steps 1 to 8 of [section 5.2](https://www.pololu.com/docs/0J69/all#5.2) - "programming using the Arduino IDE" - in the user guide.

Then connect the board via USB and then select it via Tools / Port in the IDE.

On powering it up the board this way for the first time the green USB power LED came on and the yellow user LED blinked - so it seems to come preloaded with the blink example described in step 10.

But before we upload a sketch some setup may be required on Linux to make the port accessible (if you're not already a member of the `dialout` group).

Setting up the port on Linux
----------------------------

Plug in the board and then:

    $ tail -f /var/log/syslog

Then click the board's reset button (front right) **twice**. You'll see something a little strange happen. First you'll see something like this:

    May 11 18:24:13 my-machine kernel: [5502935.227952] usb 3-4.1: USB disconnect, device number 49
    May 11 18:24:14 my-machine kernel: [5502936.193922] usb 3-4.1: new full-speed USB device number 50 using xhci_hcd
    May 11 18:24:14 my-machine kernel: [5502936.287676] usb 3-4.1: New USB device found, idVendor=1ffb, idProduct=0101
    May 11 18:24:14 my-machine kernel: [5502936.287678] usb 3-4.1: New USB device strings: Mfr=2, Product=1, SerialNumber=0
    May 11 18:24:14 my-machine kernel: [5502936.287680] usb 3-4.1: Product: Pololu A-Star 32U4 Bootloader
    May 11 18:24:14 my-machine kernel: [5502936.287681] usb 3-4.1: Manufacturer: Pololu Corporation
    May 11 18:24:14 my-machine kernel: [5502936.287842] usb 3-4.1: ep 0x82 - rounding interval to 1024 microframes, ep desc says 2040 microframes
    May 11 18:24:14 my-machine kernel: [5502936.288277] cdc_acm 3-4.1:1.0: ttyACM0: USB ACM device
    May 11 18:24:14 my-machine mtp-probe: checking bus 3, device 50: "/sys/devices/pci0000:00/0000:00:14.0/usb3/3-4/3-4.1"
    May 11 18:24:14 my-machine mtp-probe: bus: 3, device: 50 was not an MTP device

The yellow LED will fade in and out fairly quickly - this indicate the device is in bootloader mode. Then about 8 seconds later you'll see further output:

    May 11 18:24:22 my-machine kernel: [5502944.187490] usb 3-4.1: USB disconnect, device number 50
    May 11 18:24:22 my-machine kernel: [5502944.389401] usb 3-4.1: new full-speed USB device number 51 using xhci_hcd
    May 11 18:24:22 my-machine kernel: [5502944.484285] usb 3-4.1: New USB device found, idVendor=1ffb, idProduct=2300
    May 11 18:24:22 my-machine kernel: [5502944.484288] usb 3-4.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    May 11 18:24:22 my-machine kernel: [5502944.484290] usb 3-4.1: Product: Pololu A-Star 32U4
    May 11 18:24:22 my-machine kernel: [5502944.484292] usb 3-4.1: Manufacturer: Pololu Corporation
    May 11 18:24:22 my-machine kernel: [5502944.485084] cdc_acm 3-4.1:1.0: ttyACM0: USB ACM device
    May 11 18:24:22 my-machine kernel: [5502944.486365] input: Pololu Corporation Pololu A-Star 32U4 as /devices/pci0000:00/0000:00:14.0/usb3/3-4/3-4.1/3-4.1:1.2/0003:1FFB:2300.0021/input/input148
    May 11 18:24:22 my-machine kernel: [5502944.541548] hid-generic 0003:1FFB:2300.0021: input,hidraw3: USB HID v1.01 Mouse [Pololu Corporation Pololu A-Star 32U4] on usb-0000:00:14.0-4.1/input2
    May 11 18:24:22 my-machine mtp-probe: checking bus 3, device 21: "/sys/devices/pci0000:00/0000:00:14.0/usb3/3-4/3-4.1"
    May 11 18:24:22 my-machine mtp-probe: bus: 3, device: 21 was not an MTP device

This output looks almost the same as the initial block but notice that the `idProduct` values are different and the product descriptions are different - first `Pololu A-Star 32U4 Bootloader` and then `Pololu A-Star 32U4`.

So the thing changes identity on moving from bootloader mode to normal mode. So given the `idVendor` value and the two `idProduct` values we can create suitable udev rules:

    $ sudo vim /etc/udev/rules.d/50-serial-ports.rules

Add the lines:

    # Pololu A-Star 32U4
    SUBSYSTEM=="tty", ATTRS{idVendor}=="1ffb", ATTRS{idProduct}=="0101", \
        SYMLINK+="pololu-a-star-32u4", MODE="0666"

    SUBSYSTEM=="tty", ATTRS{idVendor}=="1ffb", ATTRS{idProduct}=="2300", \
        SYMLINK+="pololu-a-star-32u4", MODE="0666"

    ATTRS{idVendor}=="1ffb", ATTRS{idProduct}=="0101", ENV{MTP_NO_PROBE}="1"

    ATTRS{idVendor}=="1ffb", ATTRS{idProduct}=="2300", ENV{MTP_NO_PROBE}="1"

The `SYMLINK` means you can always reference the port by the alias `/dev/pololu-a-star-32u4` or find the real port like so:

    $ ls -l /dev/pololu-a-star-32u4

The `MODE` means the permissions for the port are set so anyone can read and write to it (resolving any "Permission denied." issues). You could alternatively resolve this via groups and be less open-to-all about things.

The `MTP_NO_PROBE` tells `mtp-probe` not to probe the device to see if it's a device that supports the [media transfer protocol](https://en.wikipedia.org/wiki/Media_Transfer_Protocol) (which it doesn't).

---

Once the port is setup you can upload sketches with the IDE upload button (without first needing to manually enter bootloader mode).

As the board seems to come preloaded with Blink set for the yellow LED let's just confirm everything works by getting the greed LED on pin 30 to blink instead.

Open File / Examples / 01.Basics / Blink and add the following line above `setup()`:

    int led = 30;

Then change all the references to `LED_BUILTIN` to `led` and then press the upload button.

Note: only pin 13 (the yellow LED) seems to support `analogWrite` - so you can fade it between on and off - but the red (pin 17) and green (pin 30) LEDs just support `digitalWrite` and can only be on or off.

---

Most of the sections after 5.2 cover more obscure topics that can really be skipped:

* 5.3 programming the board with [AVRDUDE](https://www.nongnu.org/avrdude/).
* 7 the Romi 32U4 USB interface.
* 8 the A-Star 32U4 bootloader.
* 9. reviving an unresponsive Romi 32U4.

The only important remaining section is 6 - the one on the Romi 32U4 Arduino library.

In the Arduino IDE just go to Sketch / Include Library / Manage Libraries... and then search for "romi 32u4" and click install.

You can then browse the new examples that come with this library under File / Examples / Romi32U4. Or more conveniently you can browse them on GitHub [here](https://github.com/pololu/romi-32u4-arduino-library/tree/master/examples).

There are some pretty interesting examples here so it's worth looking thru them all (the one called "Demo" demos a lot of different functionality). Many of them assume the LCD is present but a few also send their output to the serial console (and you can modify ones like "Demo" to use the serial console instead of the LCD).

E.g. try out the InertialSensors example, it depends on the LSM6 library - so go back to Manage Libraries... as above and this time search for "lsm6". There are several LSM6 libraries - select and install the one from Pololu.

Once the library is installed you can open and upload the InertialSensors, then go to Tools / Serial Monitor and see the output from the sketch (assuming "9600 baud" is selected in the monitor). If you pick up and turn the board in every direction you'll see the output values changing to reflect this.

The Pololu [LSM6 Arduino library](https://github.com/pololu/lsm6-arduino) just outputs raw 16-bit values read directly from the sensor - the comment at the start of the [InertialSensors example code](https://github.com/pololu/lsm6-arduino/blob/master/examples/Serial/Serial.ino) notes how you can covert these into something more meaningful. This kind of unit conversion is discussed in the [demo sketch section](https://learn.adafruit.com/adafruit-lsm9ds0-accelerometer-gyro-magnetometer-9-dof-breakouts?view=all#load-demo-sketch-4-11) of the Adafruit tutorial for the LSM9DS0 where they introduce their [sensor driver](https://github.com/adafruit/Adafruit_Sensor) for doing this. The LSM9DS0 is basically an LSM6DS33 with a magnotometer (Adafruit have actually discontinued their LSM9DS0 breakout and replaced it with a cheaper and somwhat less accurate [LSM9DS1 one](https://www.adafruit.com/product/3387)).

You can find full documentation for the classes and functions of the Romi32U4 library [here](http://pololu.github.io/romi-32u4-arduino-library/).

Build guide
-----------

On the Pi:

    $ sudo raspi-config

Go to Interfacing Options and enable I2C, then:

    $ sudo vim /boot/config.txt

And add the line:

    dtparam=i2c_arm_baudrate=400000

Then reboot.

In the Arduino IDE go to Manage Libraries... as before and search for and install "PololuRPiSlave".

Then under File / Examples / PololuRPiSlave select the RomiRPiSlaveDemo and upload it to the control board.

Back on the Pi:

    $ sudo apt install python3-flask python3-smbus

`python3-smbus` provide Python I2C related bindings and `python3-flask` provides a mini webframework (used by Pololu to provide a web interface to the Romi control board via the Pi).

Then go to the pololu-rpi-slave-arduino-library [releases page](https://github.com/pololu/pololu-rpi-slave-arduino-library/releases) and download and unpack the latest version (at the time of writing that was 2.0.0):

    $ curl -L -s https://github.com/pololu/pololu-rpi-slave-arduino-library/archive/2.0.0.tar.gz | tar -xvzf -
    $ mv pololu-rpi-slave-arduino-library-2.0.0 pololu-rpi-slave-arduino-library
    $ cd pololu-rpi-slave-arduino-library/pi

Now to try out an example - get the Pi to tell the control board to blink all its user LEDs:

    $ python3 ./blink.py

Pretty cool. There are a few other examples - including a benchmark of the read/write speed via I2C:

    $ python3 ./benchmark.py 
    Writes of 8 bytes: 89.7 kilobits/second
    Reads of 8 bytes: 33.3 kilobits/second

If you comment out the `i2c_arm_baudrate` line added up above and reboot you can see that the benchmark does indeed lower values:

    $ python3 ./benchmark.py 
    Writes of 8 bytes: 46.1 kilobits/second
    Reads of 8 bytes: 19.8 kilobits/second

It's noticeable though that they're far away from being four times lower.

i2c-tools on the Raspberry Pi
-----------------------------

List the I2C buses:

    $ sudo i2cdetect -l
    i2c-1   i2c         bcm2835 I2C adapter                 I2C adapter

It turns out there's only one on the Pi. Now detech devices on the bus:

    $ sudo i2cdetect -y 1
         0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
    10: -- -- -- -- 14 -- -- -- -- -- -- -- -- -- -- -- 
    20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    60: -- -- -- -- -- -- -- -- -- -- -- 6b -- -- -- -- 
    70: -- -- -- -- -- -- -- --

Note that `1` is the bus number taken from `i2c-1` above - it isn't a given that the buses are numbered from 1 (e.g. they start from 0 on the Jetson TK1).

So we see two addresses:

* 0x6b is the LSM6DS33 on the control board (see `0b1101011`, i.e. 0x6b, being used in [`LSM6.cpp`](https://github.com/pololu/lsm6-arduino/blob/master/LSM6.cpp).
* 0x14 is the MCU on the control board (see `20`, i.e. 0x14, being used in [`pi/a_star.py`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/pi/a_star.py)).

Now on the Nano
---------------

The Nano refused to boot when powered via the GPIO header.

I tried switching between pin 2 and pin 4 (on both sides) for the 5V pin but this didn't make any difference.

I monitored the startup via the serial console - there's no obvious difference between a successful start via USB power and an unsuccessful start via GPIO power - the GPIO power boot gets quite far and then just suddenly stops - i.e. the shutdown looks hard and abrupt.

See down below for how to monitor the serial console.

TODO: see if one can find the reason for the shutdown, e.g. if there's not enough current available is this logged anywhere? Try monitoring the current with an Arduino wired up to a current sensor.

**Update:** the reason there wasn't enough power was stupid - I was powering the control board via a computer USB port - it's not surprising the 500mA available weren't enough for the Nano.

When I switched to powering the control board with the power adapter I'd been using for the Nano it did boot up. However I did also experience failures to boot. The power adapter is providing maximum 2A - it may be that once it's gone through the control board the curernt available to the Nano isn't enough - with batteries connected the control board should be able to provide more current (but it will also eventually need to power the motors etc.).

---

List the I2C buses:

    $ sudo i2cdetect -l
    i2c-3   i2c         7000c700.i2c                        I2C adapter
    i2c-1   i2c         7000c400.i2c                        I2C adapter
    i2c-6   i2c         Tegra I2C adapter                   I2C adapter
    i2c-4   i2c         7000d000.i2c                        I2C adapter
    i2c-2   i2c         7000c500.i2c                        I2C adapter
    i2c-0   i2c         7000c000.i2c                        I2C adapter
    i2c-5   i2c         7000d100.i2c                        I2C adapter

Unlike the Pi (with only 1 bus) there are 7 on the Nano. Let's search them all for devices:

    $ sudo i2cdetect -y 0
    Warning: Can't use SMBus Quick Write command, will skip some addresses

It then carries on but clearly skips nearly all addresses. The following shows that bus doesn't support "quick":

    $ sudo i2cdetect -F 0
    Functionalities implemented by /dev/i2c-0:
    I2C                              yes
    SMBus Quick Command              no
    ...

It turns out that none of them support "quick" and we have to force the use of "read byte" instead (despite this having issues - see the [man page](https://jlk.fjfi.cvut.cz/arch/manpages/man/community/i2c-tools/i2cdetect.8.en)).

    $ sudo i2cdetect -y -r 0
         0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
    10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    ...

    $ sudo i2cdetect -y -r 1
         0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
    10: -- -- -- -- 14 -- -- -- -- -- -- -- -- -- -- -- 
    20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    60: -- -- -- -- -- -- -- -- -- -- -- 6b -- -- -- -- 
    70: -- -- -- -- -- -- -- --              

So the devices we're interested in are on bus 1. Running `i2cdetect` on some of the remaining buses either hung or proceeded very slowly.

    $ sudo apt install python3-flask python3-smbus

    $ wget -q -O - https://github.com/pololu/pololu-rpi-slave-arduino-library/archive/2.0.0.tar.gz | tar -xvzf -

Note that the Nano doesn't have `curl` installed by default.

    $ mv pololu-rpi-slave-arduino-library-2.0.0 pololu-rpi-slave-arduino-library
    $ cd pololu-rpi-slave-arduino-library/pi

    $ python3 ./blink.py

This fails with `Permission denied` if we look at the device for `i2c-1` we can see you have to be in group `i2c` to access it:

    $ ls -l /dev/i2c-1
    crw-rw---- 1 root i2c 89, 1 May 11 20:05 /dev/i2c-1

So:

    $ sudo usermod -a -G i2c $USER

And then log out and log back in - there's no other non-hacky way to update your current groups (see this [SO question](https://superuser.com/q/272061/238591)).

Now:

    $ cd pololu-rpi-slave-arduino-library/pi
    $ python3 ./blink.py

Works perfectly. Now:

    $ python3 benchmark.py
    Writes of 8 bytes: 34.4 kilobits/second
    Reads of 8 bytes: 13.5 kilobits/second

From this we can tell the I2C bus is only running at the default 100kHz.

You can see the bus speed for the various buses like so:

    $ cat /sys/bus/i2c/devices/i2c-?/bus_clk_rate
    400000
    100000
    400000
    100000
    400000
    400000
    400000

For whatever reason buses 1 and 3 run at 100kHz while the others run at 400kHz.

    $ echo 400000 | sudo tee -a /sys/bus/i2c/devices/i2c-1/bus_clk_rate

The `tee` is just a trick to get around the inability to do `echo 400000 > /sys/bus/i2c/devices/i2c-1/bus_clk_rate` via `sudo` (see this [Unix StackExchange answer](https://unix.stackexchange.com/q/1416/111626)).

Now if we rerun `benchmark.py` we see the speed has improved:

    $ python3 ./benchmark.py 
    Writes of 8 bytes: 81.2 kilobits/second
    Reads of 8 bytes: 31.8 kilobits/second

The numbers reported by `benchmark.py` seem surprisingly variable - and even odder, increasing the number of bytes written and read (the `n` value in the Python code) doesn't seem to improve things.

You can try other clock values, e.g. `200000` and `300000`, and see that the throughput goes up gradually but for whatever reason doesn't double, triple and quadruple.

You can keep on going higher - the speed does keep on increasing slightly but beyond 900000 it stops working altogher.

400kHz is described as [fast mode](https://www.i2c-bus.org/speed/) on the [I2C Wikipedia page](https://en.wikipedia.org/wiki/I%C2%B2C) which goes on to say 400kHz is highly compatible with basically everything (something which isn't true for the even higher speed modes).

And as it's the speed suggested by Pololu it seems sensible to not go beyond this point.

TODO: I couldn't find setting the I2C clock rate documented properly anywhere. I basically found it by running `fgrep -rl 10 $(find / -name '*i2c*')` and working through the results.

Serial console
--------------

I used an Olimex [3.3V USB to TTL serial cable](https://www.olimex.com/Products/Components/Cables/USB-Serial-Cable/USB-Serial-Cable-F/) but any similar cable should do, e.g. [this one](https://www.adafruit.com/product/954) from Adafruit.

The Jetson Nano serial console is exposed via the J44 header block - I didn't spot it immediately but the pin assignment are silkscreened onto the _underside_ of the development board.

The pin out is as follows:

* CTS
* TXD
* RXD
* NC
* RTS
* GND

With CTS being at the top (the end nearest the top edge of the board).

You don't need to use hardware flow control, so I just connected TXD, RXD and GND (as described in the FAQ section of the Olimex product page - green to TXD, red to RXD and blue to GND). Similar instructions for the Adafruit cable can be found [here](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable/connect-the-lead) (white goes to TXD, green to RXD and black to GND).

I tried using `minicom` as described in this [JetsonHacks page](https://www.jetsonhacks.com/2019/04/19/jetson-nano-serial-console/) on using the serial console - I turned off HW flow control, turned on SW flow control and set the port to 115200 8N1 as described. However no matter what I tried I got no data - there were no signs of life, not even garbage (as one often sees when the settings are wrong).

Instead I used `screen` which worked perfectly without any special configuration:

    $ screen /dev/ttyUSB0 115200

Note: I have a udev rule setup for the Olimex cable - without this you will need to run `screen` using `sudo` in order to access `/dev/ttyUSB0`.

Device tree
-----------

**Update:** this section is redundant as I discovered how to set the I2C clock speed without having to configure the I2C device tree entry as described [here](http://www.chip-community.org/index.php/Troubleshooting#I2C_.2F_TWI_problems) and other places.

    $ sudo apt install device-tree-compiler
    $ cd /boot
    $ dtc -I dtb -O dts tegra210-p3448-0000-p3449-0000-a02.dtb | less

Then search for `i2c1`, i.e. the bus we're interested in, and you'll see something like:

    i2c1 = "/i2c@7000c400";

Now search for the value on the left (without the slash), i.e `i2c@7000c400` and you'll find something like:

    i2c@7000c400 {
            #address-cells = <0x1>;
            #size-cells = <0x0>;
            compatible = "nvidia,tegra210-i2c";
            reg = <0x0 0x7000c400 0x0 0x100>;
            interrupts = <0x0 0x54 0x4>;
            iommus = <0x32 0xe>;
            status = "okay";
            clock-frequency = <0x186a0>;
            ....


The `clock-frequency` value is 0x186a0, i.e. 100,000 in decimal. For some of the other bus numbers it's 0x61a80, i.e. 400,000.

**Note:** there are various `.dtb` files in `/boot` - I just chose `tegra210-p3448-0000-p3449-0000-a02.dtb` as it was the newest but I don't know which one is appropriate.

TODO: so why is `i2c1` - the bus we see via the GPIO header - run at 100,000 and how do we change this?

Assembly
--------

1. Attach low-profile headers and buzzer.

There are two options for mounting the low-profile female headers for connecting the encoders - an inner and an outer row of holes on either side.

It doesn't make a big difference which you choose - it just affects whether you solder the corresponding male headers to the outer or inner side of the encoders themselves.

The Pololu blog suggests using the inner row on the basis that it makes connecting a HDMI connector to a Pi, that's mounted on top of the control board, easier to connect.

So we'll do this as there's no obvious better reason to favor inner over outer. However as we're not using a Pi it's not actually relevant - and anyway to get at the Pi HDMI connector while it's mounted would mean detaching the motor connectors first, so if I have to detach something why not the Pi itself (so making all its connectors easily accessible).

Taping the components to the board before soldering was a good suggestion from the blog. I still managed (twice) to end up with header not quite perpendicular.

Obvious tip: start with the first and last pin and apply the soldering iron tip to each pin such that it doesn't push the header above out of the perpendicular.

I.e. don't push from the side like this:

          +-+-+-+-+-+ Bad
          ^
          |
          |

Instead apply the iron in this direction:

    ----> +-+-+-+-+-+ Good

2. Insert battery connectors

The chassis comes with a full set of connectors (while the control board comes with just the connectors that end up soldered to it - we won't need these spare connectors).

3. Screw on the control board - this is moderately fiddly but once the first bolt is in place the rest are easy - nicely they provided one space nut and bolt in case one slips off into the distance.

4. Solder the four battery tabs to the control board - I hate these larger items that dissipate a lot of heat - make sure you have at a soldering iron with 65W or more and switch up to a larger tip.

Note: you forgot the second pair initially so the photos are out of order with point 5.

5. Solder the encoders to the motors - silkscreened side facing outwards. The motor connectors are flexible so the encoders move a little even when soldered down.

6. Snap in the motor bracks from below.

Aside: at this late stage I found I'd spent so much time working on the encoder headers, trying to get them perfectly perpendicular, that solder had flowed in and filled up the bases of the female pins. This made the header useless.

Removing the board was fairly easy. Then I used a solder sucker to get rid of as much solder as possible from the pins. It proved impossible to remove enough that it was possible to waggle the header off.

Instead I broke up the plastic of the header with my [flush cutters](https://www.adafruit.com/product/152) - it broke up very easily - and then wiggled each of the pins individually.

At first I tried pulling the pins out with a pliers while applying heat of the otherside with the soldering iron, but the pliers just sucked away all the heat.

Using my fingernails instead proved much better (and wasn't sore - clear nails are very poor conductors of heat).

Luckily both the control board and the encoders both came with a pair of low-profile headers - so I had two spares. Soldering on the replacement header proved easy - which just shows you just need a little practise before you're back in the flow.

Mounting the board back on was harder than getting it off - I tried pushing down the springs on the battery connectors with various things, but in the end sliding in my little finger did it fairly easily.

7. Slide in the motors - before they're all the way in pop on the right angle male headers and then slide the motors all the way in, with the male header of the encoders fully inserted into the female headers of the control board.

8. Solder the male headers onto the encoders - doing this in-place is the only way to ensure everything lines up - and as it turns out it's not to tricky, nothing hinders your access with the soldering iron.

9. Now push on the encoder disks. **Important:** don't push the encoder disks all the way on, so they end up flush against the encoder itself, they should be pushed onto the shaft but not enough that the shart protrudes out the other side (see the photos).

That's all the soldering finished.

10. Now install the ball caster in the rear socket (the one that's firmly part of the chassis, as opposed to the front one which is supported by a flexible arm).

**Update:** a front caster is also needed - after trying it out without one I found that the whole setup really is quite heavy and when it comes to a sudden halt it would tip forward onto the front edge with a bang. The front socket is only attached to the overall chassis by a thin arm - this looks like the least robust bit of the thing, so be careful installing the front caster (though if you mess things up a replacement [chassis base plate](https://www.pololu.com/product/3516) does just cost US$8). Without a rubber band on the top side of the socket (see the [assembly guide](https://www.pololu.com/docs/0J68/all#4) for more details) it's too flexible (on sudden stops the top of the socket ends up banging off the bottom of the control board). I used a 22x3mm nitrile o-ring (you should be able to pick them up in a hardware store) - it was quite hard to get on but proves to be the perfect size (though at 3mm it's a little too fat - 2.5mm would be better).

11. Put the tires on the wheels and attach the wheels to the motor shafts.

12. Load the battery compartment.

13. Attach it to a Pi.

14. Press the power button - if all goes well the blue power LED goes on, the buzzer beeps (this presumably depends on the current sketch?) and the Pi powers up.

Testing
-------

Now to try it out...

    $ ssh pi@raspberrypi.local
    $ cd pololu-rpi-slave-arduino-library/pi
    $ python3 ./blink.py 

If the user LEDs blick then all is good - and it's time to try controlling the Romi via the web:

    $ python3 ./server.py    
     * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
     * Restarting with stat
     * Debugger is active!
     * Debugger pin code: 260-223-657

It took several seconds for the first two lines to appear and several more for the debugger related ones.

The debugger lines are from [Werkzeug](https://werkzeug.palletsprojects.com/), one of the dependencies of [Flask](http://flask.pocoo.org/) - it's Flask that provides the web server capabilities of `server.py`. You can't actually use the PIN for anything unless something goes wrong in the Python code and you get e.g. a [500 error](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500) in your browser - with debug enabled you'll see a nice stack trace and the option to open an interactive debug console (see [here](https://werkzeug.palletsprojects.com/en/0.15.x/debug/) for more details). Unfortunately none of this is available if the errors are swallowed by AJAX (as happens with `server.py`).

You can disable all the debug related stuff by commenting out the line `app.debug = True` in `server.py` (with a `#`).

Now to access the web application being run by `server.py` open <http://raspberrypi.local:5000> in your browser. It's laid out to work well on a smartphone but works well on anything.

Note: my Android phone doesn't seem to use [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) so I had to use an IP address rather than the name `raspberrypi.local`.

Click on the LED checkboxes, press Play, then, for the exciting bit, drag your finger around on the grey motor area (or click and drag your mouse cursor).

I managed a couple of times to get things to a point where it beeped, the motors stopped and there was no way to get them going again without restarting `server.py`.

TODO: next time see if reloading the page does the trick, without restarting `server.py`. What's making the beep - it's not something obviously triggered from `server.py` and it's not something obvious in `RomiRPiSlaveDemo.ino`.

If I restarted `server.py` it turned out to be best to also reload the browser web page - otherwise odd things would happen, e.g. the motors wouldn't return to stopped on release my finger/mouse from the motor area.

It's a bit shocking how simple the code for `server.py` and `RomiRPiSlaveDemo.ino` are, just take a look:

* On the Pi: [`server.py`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/pi/server.py), [`index.html`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/pi/templates/index.html), [`main.css`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/pi/static/main.css) and [`script.js`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/pi/static/script.js).
* On the control board: [`RomiRPiSlaveDemo.ino`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/examples/RomiRPiSlaveDemo/RomiRPiSlaveDemo.ino).

The [`a_star.py`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/pi/a_star.py) library that `server.py` depends on is also tiny - it just exchanges bytes with the control board via I2C. The corresponding [TWI library](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/src/PololuTWISlave.cpp) on the control board side is also tiny.

Notes:

* Two Wire Interface (TWI) is the same as I2C (which is a registered trademark of Philips).
* The TWI library includes a [workaround](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/src/PololuRPiSlave.h) for a bug in the Pi processor's I2C implementation. See the related Raspberry Pi [bug](https://github.com/raspberrypi/linux/issues/254) - it's marked as closed as there's a workaround, not because it has been fixed. The same bug seems to exist in a processor that the Pi has used (from the original BCM2835 to the BCM2837B0 of the Pi 3 Model B+).

TODO: if you look in [`RomiRPiSlaveDemo.ino`](https://github.com/pololu/pololu-rpi-slave-arduino-library/blob/master/examples/RomiRPiSlaveDemo/RomiRPiSlaveDemo.ino) you'll see the line:

    PololuRPiSlave<struct Data,5> slave;

The `5` is a delay used to workaround the BCM283* I2C bug. For the Jetson Nano it's probably best to set this value to 0.

OK - enough about the I2C bug - back to the Pi and control board code being so small...

The real work is actually all in the separate [romi-32u4-arduino-library](https://github.com/pololu/romi-32u4-arduino-library) that provides all the classes needed to interface with the motors, encoders, buttons, LEDs and buzzer. And don't forget there's a separate library (that isn't pulled in for this setup) for the LSM6DS33.

A better IDE
------------

Download Sloeber, an Eclipse based IDE for Arduino, for [Linux](http://eclipse.baeyens.it/stable.php?OS=Linux), [Mac](http://eclipse.baeyens.it/stable.php?OS=MacOS) or [Windows](http://eclipse.baeyens.it/stable.php?OS=Windows).

Unpack the downloaded bundle and launch the IDE:

    $ tar -xf ~/Downloads/V4.3.1_linux64.2018-10-10_08-21-58.tar.gz
    $ mv sloeber sloeber-4.3.1
    $ cd sloeber-4.3.1
    $ ./sloeber-ide

Wait for it to install various bits and pieces that it needs, then restore the Welcome view (i.e. reduce from maximized).

Go to Windows / Preferences, expand Arduino, select _Third party index url's_, add `https://files.pololu.com/arduino/package_pololu_index.json`, in the main text field below the existing URLs, and click Apply.

Now, just above, go to _Platforms and Boards_, expand pololu-a-star, then expand _Pololu A-Star Boards_, select the latest version (4.0.2 at the time of writing) and click Apply.

Now, just above again, go to _Library Manager_, type romi32u4 into the search field and tick the checkbox beside the displayed library, then search for PololuRPiSlave and do the same.

Finally click _Apply and Close_.

Connect the control board via USB, click the _Create new sketch_ button, then:

* Enter a project name, e.g. romi-jetson-nano.
* From the platform folder dropdown select the Pololu one, for board select _Pololu A-Star 32U4_, then select the appropriate port, e.g. `/dev/ttyACM0`.
* From the select code dropdown select _Sample sketch_, below expand Library, then PololuRPiSlave and tick RomiRPiSlaveDemo.
* Click Finish.

Now you can expand the project in the _Project Explorer_ view, go to `RomiRPiSlaveDemo.ino`, now you can control-click on things in the source like `PololuRPiSlave`.

Make sure to click the _Link with Editor_ button in the _Project Explorer_ view so that this view stays in sync with the currently selected editor tab.

Power mode
----------

By default there's no constraint on how much power the Nano module can ask for. The module itself can consume up to 2A and this is the maximum that can be provided via the micro-USB power connector.

This can be problematic is the development board also has to power additional hardware such as a camera.

You can switch the module into a lower power mode so that it consumes at most 1A:

    $ sudo nvpmodel -m 1

The values, e.g. 1 here, are defined in `/etc/nvpmodel.conf`, search for `POWER_MODEL` and you'll find (at least) two definitions with `ID` and `NAME` attributes.

You can query the current power mode like so:

    $ sudo nvpmodel -q
    NV Power Mode: 5W
    1

And you can set it back to unconstrained power consumption (`MAXN`) like so:

    $ sudo nvpmodel -m 0
    $ sudo nvpmodel -q
    NV Power Mode: MAXN
    0

Nano control continued
----------------------

Remove the previously installed orignal `pololu-rpi-slave-arduino` library and install my fork:

    $ ssh ghawkins@JetsonNano.local
    $ rm -r pololu-rpi-slave-arduino-library
    $ git clone git@github.com:george-hawkins/pololu-rpi-slave-arduino-library.git

Make sure the relevant I2C bus is running at 400kHz:

    $ echo 400000 | sudo tee -a /sys/bus/i2c/devices/i2c-1/bus_clk_rate

Start the server:

    $ cd pololu-rpi-slave-arduino-library/pi
    $ python3 ./server.py 

In your browser open <http://jetsonnano.local:5000/>

Set I2C bus speed at startup
----------------------------

    $ ssh ghawkins@JetsonNano.local
    $ cd git/pololu-romi-jetbot
    $ sudo cp i2c-1-400kHz.service /etc/systemd/system
    $ sudo systemctl daemon-reload
    $ systemctl enable i2c-1-400kHz

See the `i2c-1-400kHz.service` file itself for more details.

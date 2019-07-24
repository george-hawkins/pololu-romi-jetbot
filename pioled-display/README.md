PiOLED display
==============

To display a piece of text:

    $ cd ~/git/pololu-romi-jetbot/pioled-display
    $ ./text-to-bmp '1 2 3'
    $ ./pioled-display tmp/tmp.tJubLaZTD7.bmp

Installing a NetworkManager script
----------------------------------

To display the IP address associated with `wlan0` (or `eth0` if an address for `wlan0` isn't available).

    $ cd ~/git/pololu-romi-jetbot/pioled-display
    $ file=99-ip-to-pioled
    $ echo '#!/bin/bash' > $file
    $ echo $(readlink -f ip-to-pioled) >> $file
    $ chmod a+x $file
    $ cat $file
    #!/bin/bash
    /home/ghawkins/git/pololu-romi-jetbot/pioled-display/ip-to-pioled
    $ sudo cp $file /etc/NetworkManager/dispatcher.d && rm $file

Note: this script assumes that you're using IPv4 and that the interfaces are `eth0` and `wlan0`.

For more about network services, see ["network services with NetworkManager dispatcher"](https://wiki.archlinux.org/index.php/NetworkManager#Network_services_with_NetworkManager_dispatcher).

Upgrading to Blinka
-------------------

If you look at the origin [`stats.py`](https://github.com/NVIDIA-AI-IOT/jetbot/blob/master/jetbot/apps/stats.py) you'll see the line:

    disp = Adafruit_SSD1306.SSD1306_128_32(rst=None, i2c_bus=1, gpio=1) # setting gpio to 1 is hack to avoid platform detection

Why do we need this hack to avoid platform detection? If you remove `gpio=1` the script fails with:

    "/usr/local/lib/python3.6/dist-packages/Adafruit_GPIO-1.0.4-py3.6.egg/Adafruit_GPIO/GPIO.py", line 426, in get_platform_gpio
    ModuleNotFoundError: No module named 'Jetson'

I.e. the method `get_platform_gpio` in `Adafruit_GPIO/GPIO.py` doesn't know about Jetson.

This is odd as Adafruit announced [Blinka support for Jetson](https://blog.adafruit.com/2019/03/18/adafruit-blinka-support-for-the-nvidia-jetson-series-nvidia-gtc19-nvidiaembedded/).

However all becomes clear if you look at [`Adafruit_Python_GPIO`](https://github.com/adafruit/Adafruit_Python_GPIO) - this is a deprecated pre-Blinka version of the library.

So let's create a new version of `pioled-display` that uses uses Blinka...

    $ sudo apt install python3-venv
    $ python3 -m venv env
    $ source env/bin/activate

To leave a virtual environment:

    $ deactivate 

adafruit-circuitpython-ssd1306 depends on wheel but omits to specify it as a dependency.

    $ sudo apt install python3-wheel

    $ pip3 install adafruit-circuitpython-ssd1306

By default gpio is only accessible by root:

    $ l /sys/class/gpio/export 

You have to follow the instructions from <https://pypi.org/project/Jetson.GPIO/> like so:

    $ sudo groupadd -r gpio
    $ sudo usermod -a -G gpio $USER
    $ exit

Relogin and check you're now in the `gpio` group:

    $ groups
    ... gpio

Carry on with the instructions:

    $ sudo cp /opt/nvidia/jetson-gpio/etc/99-gpio.rules /etc/udev/rules.d

The Jetson.GPIO page says you can reboot or instead do:

    $ sudo udevadm control --reload-rules
    $ sudo udevadm trigger
    $ l /sys/class/gpio/export 

But I found I had to reboot:

    $ sudo reboot now

Relogin and ...

    $ l /sys/class/gpio/export 
    $ cd ~/git/pololu-romi-jetbot/pioled-display
    $ source env/bin/activate

    $ pip install Pillow

    $ mv pioled-display old-pioled-display
    $ vi pioled-display
    $ chmod 755 pioled-display 
    $ ./text-to-bmp 'a b c'
    $ ./pioled-display /tmp/tmp.JQZ59Ovntz.bmp

The run time for `pioled-display` was much longer than expected. I found creating `SSD1306_I2C(128, 32, i2c)` was what slowed everything down.

After some experimentation I found that `cProfile` and `gprof2dot` were the things to use.

    $ cat > foo-test << EOF
    #!/usr/bin/env python3

    from board import SCL, SDA
    from busio import I2C
    from adafruit_ssd1306 import SSD1306_I2C

    import cProfile

    i2c = I2C(SCL, SDA)

    cProfile.run("SSD1306_I2C(128, 32, i2c)", 'foo.profile')
    EOF
    $ chmod 755 foo-test

    $ ./foo-test 

    $ pip install gprof2dot
    $ gprof2dot -f pstats foo.profile -o call-graph.dot

From my main machine:

    $ scp ghawkins@jetsonnano.local:/home/ghawkins/git/pololu-romi-jetbot/pioled-display/call-graph.dot .
    $ xdot call-graph.dot

From this I could see that the problem seemed to be the `writeto` in [`i2c_device.py`](https://github.com/adafruit/Adafruit_CircuitPython_BusDevice/blob/master/adafruit_bus_device/i2c_device.py):

    while not i2c.try_lock():
        pass
    try:
        i2c.writeto(device_address, b'')
    except OSError:
        # some OS's dont like writing an empty bytesting...
        # Retry by reading a byte
        try:
            result = bytearray(1)
            i2c.readfrom_into(device_address, result)
        except OSError:
            raise ValueError("No I2C device at address: %x" % device_address)
    finally:
        i2c.unlock()

The odd thing is that the profiler output says `writeto` is called 34 times but I don't really see how ends up being called more than once.

I isolated this into `foo-test` and indeed it performed slowly as expected, then I ran it again and it then worked almost instantaneously.

It and the original `pioled-display` then continued to run almost instantaneously after that. I'd seen similar behavior with the pre-Blinka `stats.py` script as well, where there was an unexplained pause on startup that went away eventually. So this odd behavior seems to be common to both.

I didn't investigate further.

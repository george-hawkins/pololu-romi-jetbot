Using the camera module with the Raspberry Pi
=============================================

IP addresses
------------

The Raspberry Pi advertises its IP address on your local network using [Avahi](https://en.wikipedia.org/wiki/Avahi_(software)) - so you can generally connect to it using the name `raspberrypi.local` rather than a raw IP address like 192.168.0.66. I've used `raspberrypi.local` all through this page but you _may_ need to use the actual IP address of your Pi. There are no end of ways of determining this, e.g. see this [SO answer](https://stackoverflow.com/questions/13322485/how-to-get-the-primary-ip-address-of-the-local-machine-on-linux-and-os-x). The most common suggestion is:

    $ ifconfig
    eth0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            ether b8:27:eb:a0:96:ba  txqueuelen 1000  (Ethernet)
            ...

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            ...

    wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.0.66  netmask 255.255.255.0  broadcast 192.168.0.255
            ...

And look for `inet` either under `wlan0` if you're using WiFi (as above) or under `eth0` if you're using ethernet.

Below I'm always ssh-ed into the Raspberry Pi from my laptop, so `raspberrypi.local` is the name of the remote Pi and wherever you see `$MY_IP_ADDR` that's the address of my laptop, i.e. my local machine. So you need to replace `$MY_IP_ADDR` with the address of your local machine or first set `MY_IP_ADDR` to the address of your machine:

    $ MY_IP_ADDR=192.168.0.241

Obviously replace `192.168.0.241` with the value appropriate to your machine.

Connecting the camera module
----------------------------

Pull up the top of the connector and push it backwards (it can only be pushed one direction - lets call that back).

Insert the cable so that the shiny metal connectors at its end are facing forward and the other side of the cable is the side facing the top of the connector that you just pulled up and back.

Now push the top of the connector back into place, boot the Pi and run:

    $ sudo raspi-config

Select Interfacing Options, then select the Camera and go thru the obvious steps to enable it.

Various pages note that `gpu_mem` must be at least 32M with 128M recommended - if you look in `/boot/config.txt` you see that 128M is the pre-configured default. So this hardly seems worth mentioning but maybe at one stage the default for `gpu_mem` was lower. TODO: check if `gpu_mem` is maybe updated to 128M from a lower value when setting up the camera with `raspi-config`.

Camera modes and FoV
--------------------

Remember that the [Waveshare fisheye camera](https://www.waveshare.com/rpi-camera-g.htm) uses the older OV5647 sensor of the version 1 Raspberry Pi camera.

So it has the modes listed for the OV5647 in Raspberry Pi [documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).

But the Picamera (python library) documentation has a much better [overview](https://picamera.readthedocs.io/en/release-1.12/fov.html) or the modes with pictures showing the FoV of the older OV5647 sensor and the newer IMX219 sensor.

Raspivid overlay
----------------

You can get `raspivid` to overlay values like the current time using the `-a` flag. To overlay all possible values use `-a 1023`:

    $ raspivid -a 1023 ...

For the individual values, e.g. 64 for gain settings, see the Pi [camera documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).

Streaming using raspivid
------------------------

Initially I wanted to use V4L (see the later sections covering its usage) but streaming with `raspivid` proved far less error prone than using V4L - it could be killed and restarted repeatedly without any issues.

On the Pi:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -l -o tcp://0.0.0.0:9090

Options:

* `-v` - verbose.
* `-n` - disable preview window.
* `-t 0` - start capturing immediately.
* `-md 6 -w 640 -h 480` - mode 6 with width and height set to match the sensor mode.
* `-fps 42` - frames-per-second matching the lower range of mode 6.
* `-vf` - vertical flip (if you appear upside-down).
* `-fli auto` - avoid flicker.
* `-l -o tcp://0.0.0.0:9090` - listen for TCP connections on port 9090.

See the Picamera [hardware page](https://picamera.readthedocs.io/en/release-1.12/fov.html) for the list of modes and diagrams of what this means it terms of FoV (which the otherwise comprehensive Raspberry Pi [camera documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) doesn't have). The width and height here correspond to mode 6 on the V1 camera, on the newer V2 camera the corresponding mode would be `-md 7 -w 640 -h 480`.

On the local machine:

    $ mplayer -nolirc -fps 200 -demuxer h264es ffmpeg://tcp://raspberrypi.local:9090

Usually I use VLC but it introduces a significant lag. To see this restart the Pi side and instead of `mplayer` use:

    $ cvlc tcp/h264://raspberrypi.local:9090

Presumably the lag can be tuned with the appropriate command line arguments?

An FPS of 200 sounds extreme but it doesn't seem to mean what is seems. Leaving it at a far higher value than the FPS than the Pi is producing is fine but lower it too far and lag suddenly appears, so one might as well set it higher than the highest possible Pi FPS.

TODO: I'd like to know what's going on here - it I set both sides to 30 FPS I get serious lag, if I set them both to 42 (with mode 6) then all seems fine but I don't gain anything in quality or reduced CPU usage by setting MPlayer's FPS lower - so it seems sticking with 200 is simplest.

Oddly no matter what the setting lag seems to appear and disappear over time even if there's no lag initially.

Note: `9090` is a randomly used port number, I use it consistently here but one can chose any number above 1023 (the ports below are reserved for privileged services) and below 65536 and which aren't already being used by something else (actually the range 1024 to 49151 is best as ports above 49151 are used for dynamic ports so they may randomly be taken at any given time).

Flicker avoidance
-----------------

Very few examples I've found of `raspivid` usage use the flicker avoidance flag `-fli` but I found it dramatically reduced the flicker I saw when using the camera. The flicker avoidance flag was introduced relatively [late](https://github.com/raspberrypi/userland/pull/406) and for whatever isn't documented on the [camera.md](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) page (I've submitted a [GitHub issue](https://github.com/raspberrypi/documentation/issues/1134) regarding this).

Binning
-------

The camera sensor supports a number of modes with various resolutions and frame rates. In these examples I use a very low resolution - 640x480 - but if you look at the Picamera [hardware page](https://picamera.readthedocs.io/en/release-1.12/fov.html) you'll notice that the 640x480 sensor mode involves binning (4x4 binning with the V1 sensor and 2x2 binning with the V2 sensor).

[Binning](https://www.ubergizmo.com/what-is/pixel-binning-camera/) that multiple sensor pixels are combined to form one super pixel to reduce noise. This can be useful in low light situations (but whether it outways the loss in resolution depends on the situation).

Anyway 640x480 was chosen for these examples just because it was convenient to work with small video windows when experimenting with various settings etc. and not because of binning.

Hardware limitations
--------------------

The [camera.md page](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) notes that H.264 (as used in most of the examples here) can't support greater than 1080p (see the `--codec` flag section) and that you have to use the MJPEG codec to encode up to the full sensor size (but that this results in reduced frame rates and increased CPU usage as H.264 has hardware support and MJPEG does not). The [hardware limits section](https://picamera.readthedocs.io/en/release-1.12/fov.html#hardware-limits) of the Picamera hardware page notes the same thing - they talk about a horizontal resolution of 1920 which is the same thing as 1080p (i.e. 1920x1080 px).

So with the V2 camera module you can't use the full sensor unless you use MJPEG, the nearest you can get is 1640x922 which omits a band at the top and the bottom of the sensor.

Note: I link to the 1.12 version of the Picamera hardware page here. The current version (1.13 at the time of writing) features a much expanded hardware page but the basics, covered here, get lost way down the page.

Netcat
------

Programs like `raspivid` and `mplayer` can interact with the network directly but the URIs they use are fairly cryptic and are application specific, e.g. `tcp://0.0.0.0:9090` above for `raspivid` and `ffmpeg://tcp://raspberrypi.local:9090` for `mplayer`.

Some programs can't interact with the network directly and can only deal with normal input and output - with these programs you can use tools like [netcat](https://en.wikipedia.org/wiki/Netcat) to enable them to communicate across a network. Once you know the necessary netcat commands you can use them with _every_ tool - it's even convenient to use them with tools like `raspivid` and `mplayer` rather than working out their cryptic URI formats.

So you can get `raspivid` and `mplayer` to communicate with each other via netcat like so:

First on your local machine start netcat so it will listen for a connection from you Pi and pipe the resulting data to MPlayer:

    $ nc -l 9090 | mplayer -nolirc -fps 200 -demuxer h264es -

Then on the Pi start `raspivid` and pipe its output to netcat such that it forwards it to your local machine:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -o - | nc $MY_IP_ADDR 9090

In the above setup it's the MPlayer side that waits (and must be started first). You can switch it around and start the Pi side first such that it waits for a connection from the local machine:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -l -o - | nc -l 9090

And then start MPlayer on your local machine:

    $ nc raspberrypi.local 9090 | mplayer -nolirc -fps 200 -demuxer h264es -

Important: notice the additional `-l` flag passed to `raspivid`. In the first setup above no flags were needed to tell `raspivid` or `mplayer` that any other programs were involved. Here `raspivid` has to know not to start producing data immediately (if it does things back up almost instantly and the pipeline exits) - and that's what the `-l` (listen) flag tells it.

The nice thing with this setup though is that you can tell netcat on the Pi side not to exit when MPlayer is closed or otherwise stops consuming data. With `-k` you can tell it to listen for another connection once the current one dies:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -l -o - | nc -k -l 9090

GStreamer
---------

[GStreamer](https://en.wikipedia.org/wiki/GStreamer) is a multimedia framework that on other systems, in combination with V4L, is often used to do the kind of things we've been doing here with `raspivid`. GStreamer is a set of libraries and tools - here we'll use some of those tools in combination with `raspivid` on the Pi side to stream video to GStreamer tools on your local machine.

Install GStreamer on the Pi as per the GStreamer [installation guide](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html). Oddly there isn't one single package that pulls in all necessary dependencies. Instead you install a long list of dependencies - on trying to install all of them on the Pi it tells you that some of them don't exist (`gstreamer1.0-qt5`, `gstreamer1.0-gtk3` and `gstreamer1.0-gl`) so just leave them out and try again.

Then on the Pi to stream data via `gst-launch-1.0`:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -o - | gst-launch-1.0 fdsrc !  h264parse ! rtph264pay config-interval=1 pt=96 ! udpsink host=$MY_IP_ADDR port=9090

On the local machine:

    $ gst-launch-1.0 udpsrc port=9090 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false

This example uses UDP which has two nice properties:

* The Pi side can just start streaming even if there's no one on the other side listening yet.
* The client can be killed and restarted repeatedly without the Pi side being aware that anything has happened.

You can avoid the complex `caps` specification by using the GStreamer elements `gdppay` and `gdpdepay` but this requires a TCP connection (with the Pi side started first. On the Pi:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -o - | gst-launch-1.0 fdsrc !  h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=0.0.0.0 port=9090

On the local machine:

    $ gst-launch-1.0 tcpclientsrc host=raspberrypi.local port=9090 ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false

GStreamer has changed a lot over time and many examples on the web involve elements that have been renamed or replaced. To find missing elements:

* Some have just been completely renamed, e.g. ffmpegcolorspace to videoconvert - you just have to google for these.
* Most older elements starting with "ff", e.g. ffdec_h264, have been renamed to "av", so ffdec_h264 became avdec_h264
* You can grep for likely elements with like so `gst-inspect-1.0 | fgrep 264`

UDP
---

The first GStreamer example above uses UDP. I tried using UDP with `raspivid` directly but didn't get such nice results. First `raspivid` would close unless there was something ready to receive and second I couldn't get MPlayer to work properly with the UDP stream, VLC worked but introduced a significant lag.

First on the local machine:

    $ mplayer -nolirc -fps 200 -demuxer h264es ffmpeg://udp://@:9090

Then on the Pi:

    $ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 42 -fli auto -vf -o udp://$MY_IP_ADDR:9090

The lower half of the image appears to be melting but there is no significant lag. Now with VLC on the local machine:

    $ cvlc udp/h264://@:9090

Then on the Pi the same as above. This time the video looks good but there is a significant lag.

V4L on Raspberry Pi
-------------------

[Video4Linux](https://en.wikipedia.org/wiki/Video4Linux) (V4L) is a standard API for interacting with video devices and it would be nice to use it and avoid Raspberry Pi specific tools like `raspivid`. V4L was not available on the Raspberry Pi when it was first launched (and tools like `raspivid` do not use it) but support for it was introduced [eventually](https://www.raspberrypi.org/forums/viewtopic.php?t=62364). My experience though with using V4L on the Raspberry Pi was that it was fairly flakey and `raspivid` behaved far more consistently, without error or problems.

However here are my notes on using it - V4L is at version 2, so everywhere you'll see V4L2 from now on.

By default the V4L2 module is not loaded and there is no corresponding `/dev/video0` device. Loading the module (and creating the device) you need to do:

    $ sudo modprobe bcm2835-v4l2

Then you can display all supported video formats including frame sizes:

    $ v4l2-ctl --list-formats-ext

And then list the supported FPS range for a given format and dimensions:

    $ v4l2-ctl --list-frameintervals=width=2592,height=1944,pixelformat=BGR4

Note: `cvlc` is VLC without a UI.

To stream:

    $ cvlc v4l2:///dev/video0 --v4l2-width 1920 --v4l2-height 1080 --v4l2-chroma h264 --sout '#standard{access=http,mux=ts,dst=:9090}'

Among other output you see:

    [0175f108] vlcpulse audio output error: PulseAudio server connection failure: Connection refused
    [01758840] dbus interface error: Failed to connect to the D-Bus session daemon: Unable to autolaunch a dbus-daemon without a $DISPLAY for X11
    [016fe110] main libvlc error: interface "dbus,none" initialization failed

It's possible to get rid of the PulseAudio audio error with `--aout dummy` and the D-Bus error with `--no-dbus`. So you end up with:

    $ cvlc --aout dummy --no-dbus v4l2:///dev/video0 --v4l2-width 1920 --v4l2-height 1080 --v4l2-chroma h264 --sout '#standard{access=http,mux=ts,dst=:9090}'
    VLC media player 3.0.6 Vetinari (revision 3.0.6-0-g5803e85f73)
    [00103288] main interface error: no suitable interface module
    [0009a110] main libvlc error: interface "globalhotkeys,none" initialization failed
    [001032f0] dummy interface: using the dummy interface module...

After much searching there seems to be no way to get rid of the `main interface error` and the `main libvlc error` - they seem to be just warnings rather than errors.

So if all has gone well this will stream video on port 9090.

Then on the receiving machine you can start VLC, go to Media / Open Network Stream... and enter the URL `http://raspberrypi.local:9090`.

The video stream takes a few seconds to start playing and then lags behind reality by a similar amount of time.

Totem, the default player on Ubuntu, works just as well - as do presumably a plethora of other Linux media players. Note that totem uses GStreamer under the covers.

Often on killing and restarting `cvlc` I'd see the error:

    [0071a0b8] v4l2 demux error: cannot start streaming: Operation not permitted
    [0071a0b8] v4l2 demux error: not a radio tuner device

The seems to be a consequence of VLC not releasing the `/dev/video0` device properly. The only solution to this seemed to be to remove and reload the underlying `bcm2835-v4l2` module like so:

    $ rmmod bcm2835-v4l2 ; modprobe bcm2835-v4l2

Extremely frequently I also saw the error:

    [0093ac18] main decoder error: buffer deadlock prevented

There seemed to be no solution to this other than to repeatedly remove and reload the `bcm2835-v4l2` module as above and restart `cvlc` until eventually it started without this error. The error seems to imply that something has been prevented (and therefore the situation is now OK) but this seems to be a serious error - no streaming occurs if this error appears.

Either my hardware or the v4l2 driver seemed to be very flakey when used like this.

V4L notes
---------

You can see the current settings that V4L2 can display like so:

    $ v4l2-ctl --all

You can see e.g. the range of values and the current value for auto-exposure:

    auto_exposure (menu)   : min=0 max=3 default=0 value=0

Some settings are marked `(int)` meaning they just take a simple integer value, e.g. brightness takes any value from 0 to 100.

For the ones that are marked `(menu)` like auto-exposure above you can see what the numerica values, in this case 0 to 3, mean like so:

    $ v4l2-ctl --list-ctrls-menus

This enumerates the menus for all `(menu)` settings, so for auto-exposure you see that 0 is 'Auto Mode' and 1 is 'Manual Mode', oddly there's no items for the additional acceptable values 2 and 3.

To see all this data presented in a UI you can use the [Video4Linux2 Universal Control Panel](http://manpages.ubuntu.com/manpages/disco/man1/v4l2ucp.1.html).

Make sure to ssh into your Pi with X11 forwarding enabled, i.e. use `-X`:

    $ ssh -X pi@raspberrypi.local
    $ sudo apt install v4l2ucp
    $ v4l2ucp /dev/video0

You'll see the control panel pop up on your local system (assuming there's a local X server). However before you get to that point it'll complain, with a long stream of popups, about situations like the one seen above where there's no menu text for the auto-exposure values 2 and 3 - with each popup saying something like ""Unable to get menu item for Auto Exposure, index=2. Will use unknown".

To be honest I find the text output of `v4l2-ctl` more digestable.

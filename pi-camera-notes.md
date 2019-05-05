Using the camera module with the Raspberry Pi
=============================================

Below I'm always ssh-ed into the Raspberry Pi from my workstation (running Ubuntu), so `raspberrypi.local` is the name of the remote Pi and `${SSH_CLIENT%% *}` is the address of my workstation, i.e. my local machine. When you ssh into the Pi the environment variable `SSH_CLIENT` is set:

    $ echo $SSH_CLIENT
    192.168.0.241 48232 22

So `${SSH_CLIENT%% *}` just strips away everything except the IP address.

The Raspberry Pi advertises its IP address on your local network using [Avahi](https://en.wikipedia.org/wiki/Avahi_(software)) - so you can generally connect to it using the name `raspberrypi.local` rather than a raw IP address like 192.168.0.66. However you _may_ need to use the actual IP address of your Pi. There are no end of ways of determining this, e.g. see this [SO answer](https://stackoverflow.com/questions/13322485/how-to-get-the-primary-ip-address-of-the-local-machine-on-linux-and-os-x).

Setting up the camera
---------------------

First connect the camera as shown in [`pi-connecting-camera.md`](pi-connecting-camera.md) then power up the Pi, ssh in and run:

    $ sudo raspi-config

Select Interfacing Options, then select the Camera and go thru the remaining obvious steps to enable it.

Various pages note that when using the camera the `gpu_mem` must be at least 32M with 128M recommended - if you look in `/boot/config.txt` you see that 128M is the pre-configured default. Maybe at one stage the default for `gpu_mem` was lower (otherwise it hardly seems worth mentioning). TODO: check if `gpu_mem` is maybe updated to 128M from a lower value when setting up the camera with `raspi-config`.

Camera modes
------------

For all the examples here I've used a [Raspberry Pi camera module V2](https://www.raspberrypi.org/products/camera-module-v2/). It uses a IMX219 sensor. The sensor has various modes and you can see these for the IMX219 in the Raspberry Pi [camera documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).

**Update:** the Picamera (python library) documentation has a much better [overview](https://picamera.readthedocs.io/en/release-1.12/fov.html) of the modes with pictures showing the FoV of the older OV5647 sensor and the newer IMX219 sensor.

In all the examples I've used mode 5 as this is the IMX219 mode with the best FoV that can still be handled by the hardware H.264 encoder.

Streaming using raspivid
------------------------

I started out using V4L (see the later sections covering its usage) with VLC but streaming with `raspivid` proved far simpler. It was only after I got used to using the camera via `raspivid` that I moved back to trying a pure GStreamer pipline with V4L. Even though I did eventually get GStreamer working well with V4L I have to say `raspivid` is still much easier to work with and provides more direct control of the camera, e.g. you can set the camera sensor modes directly (with `-md`) and set things like flicker avoidance which are not available via V4L.

Let's start straight in on setting up `raspivid` to stream on the Pi:

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -l -o tcp://0.0.0.0:9090

Options:

* `-v` - verbose.
* `-n` - disable preview window.
* `-t 0` - start capturing immediately.
* `-md 5 -w 1640 -h 922` - mode 5 with width and height set to match the sensor mode.
* `-fps 20` - frames-per-second matching the mid range of mode 6.
* `-vf` - vertical flip (if you appear upside-down).
* `-fli auto` - avoid flicker.
* `-l -o tcp://0.0.0.0:9090` - listen for TCP connections on port 9090.

See the Picamera [hardware page](https://picamera.readthedocs.io/en/release-1.12/fov.html) for the list of modes and diagrams of what this means it terms of FoV (which the otherwise comprehensive Raspberry Pi [camera documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) doesn't have). The width and height here correspond to mode 5 on the V2 camera module.

On the local machine:

    $ sudo apt install mplayer
    $ mplayer -nolirc -fps 200 -demuxer h264es ffmpeg://tcp://raspberrypi.local:9090

Usually I use VLC but it introduces a significant lag. To see this restart the Pi side and instead of `mplayer` use:

    $ cvlc tcp/h264://raspberrypi.local:9090

Presumably the lag can be tuned with the appropriate command line arguments?

An FPS of 200 sounds extreme but it doesn't seem to mean what is seems. Leaving it at a far higher value than the FPS than the Pi is producing is fine but lower it too far and lag suddenly appears, so one might as well set it higher than the highest possible Pi FPS.

TODO: I'd like to know what's going on here - if I set the MPlayer FPS below a certain seemingly arbitrary value then I start to get serious lag, if I set it to just above this value then all seems fine but I don't gain anything in quality or reduced CPU usage - so it seems simplest to stick with 200.

Oddly no matter what the setting lag seems to appear and disappear over time even if there's no lag initially.

Note: `9090` is a randomly used port number, I use it consistently here but one can chose any number above 1023 (the ports below are reserved for privileged services) and below 65536 and which aren't already being used by something else (actually the range 1024 to 49151 is best as ports above 49151 are used for dynamic ports so they may randomly be taken at any given time).

Raspivid overlay
----------------

You can get `raspivid` to overlay values like the current time using the `-a` flag. To overlay all possible values use `-a 1023`:

    $ raspivid -a 1023 ...

For the individual values, e.g. 64 for gain settings, see the Pi [camera documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).

Flicker avoidance
-----------------

Very few examples I've found of `raspivid` usage use the flicker avoidance flag `-fli` but I found it dramatically reduced the flicker I saw when using the camera. The flicker avoidance flag was introduced relatively [late](https://github.com/raspberrypi/userland/pull/406) and for whatever isn't documented on the [camera.md](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) page (I've submitted a [GitHub issue](https://github.com/raspberrypi/documentation/issues/1134) regarding this).

Binning
-------

The camera sensor supports a number of modes with various resolutions and frame rates. If you look at the Picamera [hardware page](https://picamera.readthedocs.io/en/release-1.12/fov.html) you'll see that some of these modes involve something called binning.

[Binning](https://www.ubergizmo.com/what-is/pixel-binning-camera/) means that multiple sensor pixels are combined to form one super pixel to reduce noise. This can be useful in low light situations (but whether it outways the loss in resolution depends on the situation).

Binning used to be more import with the V1 camera module that used the older OV5647 sensor, as it supported up to 4x4 binning. The newer IMX219 sensor of the V2 module only supports up to 2x2 binning.

Older OV5647 sensor
-------------------

As noted the V1 camera module uses the older OV5647 sensor and many modules from third parties continue to use this sensor, e.g. the popular range of Waveshare modules (I've got a [Waveshare fisheye camera](https://www.waveshare.com/rpi-camera-g.htm)).

For cameras using the OV5647 sensor I suggest using `-md 6 -w 640 -h 480 -fps 42` for these examples rather than the `-md 5 -w 1640 -h 922 -fps 20` seen here. Mode 6 uses the full sensor and applies 4x4 binning which works well in low-light settings. 42 FPS is the lower end of the FPS range for this mode.

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

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -o - | nc ${SSH_CLIENT%% *} 9090

In the above setup it's the MPlayer side that waits (and must be started first). You can switch it around and start the Pi side first such that it waits for a connection from the local machine:

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -l -o - | nc -l 9090

And then start MPlayer on your local machine:

    $ nc raspberrypi.local 9090 | mplayer -nolirc -fps 200 -demuxer h264es -

Important: notice the additional `-l` flag passed to `raspivid`. In the first setup above no flags were needed to tell `raspivid` or `mplayer` that any other programs were involved. Here `raspivid` has to know not to start producing data immediately (if it does things back up almost instantly and the pipeline exits) - and that's what the `-l` (listen) flag tells it.

The nice thing with this setup though is that you can tell netcat on the Pi side not to exit when MPlayer is closed or otherwise stops consuming data. With `-k` you can tell it to listen for another connection once the current one dies:

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -l -o - | nc -k -l 9090

GStreamer
---------

[GStreamer](https://en.wikipedia.org/wiki/GStreamer) is a multimedia framework that on other systems, in combination with V4L, is often used to do the kind of things we've been doing here with `raspivid`. GStreamer is a set of libraries and tools - here we'll use some of those tools in combination with `raspivid` on the Pi side to stream video to GStreamer tools on your local machine. Later we'll come to using it with V4L and without `raspivid`.

Install GStreamer on the Pi as per the GStreamer [installation guide](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html). Oddly there isn't one single package that pulls in all necessary dependencies. Instead you install a long list of dependencies - on trying to install all of them on the Pi it tells you that some of them don't exist (`gstreamer1.0-qt5`, `gstreamer1.0-gtk3` and `gstreamer1.0-gl`) so just leave them out and try again.

Then on the Pi to stream data via `gst-launch-1.0`:

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -o - | gst-launch-1.0 fdsrc !  h264parse ! rtph264pay config-interval=1 pt=96 ! udpsink host=${SSH_CLIENT%% *} port=9090

On the local machine:

    $ gst-launch-1.0 -v udpsrc port=9090 caps='application/x-rtp, media=video, clock-rate=90000, encoding-name=H264' ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false

This example uses UDP which has two nice properties:

* The Pi side can just start streaming even if there's no one on the other side listening yet.
* The client can be killed and restarted repeatedly without the Pi side being aware that anything has happened.

You can avoid the complex `caps` specification by using the GStreamer elements `gdppay` and `gdpdepay` but this requires a TCP connection (with the Pi side started first. On the Pi:

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -o - | gst-launch-1.0 fdsrc !  h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=0.0.0.0 port=9090

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

    $ raspivid -v -n -t 0 -md 5 -w 1640 -h 922 -fps 20 -fli auto -vf -o udp://${SSH_CLIENT%% *}:9090

The lower half of the image appears to be melting but there is no significant lag. Now with VLC on the local machine:

    $ cvlc udp/h264://@:9090

Then on the Pi the same as above. This time the video looks good but there is a significant lag.

V4L on Raspberry Pi
-------------------

[Video4Linux](https://en.wikipedia.org/wiki/Video4Linux) (V4L) is a standard API for interacting with video devices and using it avoid tying yourself to Raspberry Pi specific tools like `raspivid`. V4L was not available on the Raspberry Pi when it was first launched (and tools like `raspivid` do not use it) but support for it was introduced [eventually](https://www.raspberrypi.org/forums/viewtopic.php?t=62364). Note that V4L is at version 2, hence the `v4l2` seen in many places.

My initial experience of using V4L on the Raspberry Pi was marred by trying to use it with VLC. This led me to believe it was flakey and unreliable but really the problem was VLC. VLC may be an excellent media player on most platforms but it does not seem to play well with V4L and live video on Raspberry Pi. I've moved out my notes on using VLC to stream from the Pi into [`pi-vlc-v4l.md`](pi-vlc-v4l.md) but I would not recommend using it in this situation.

By default the V4L module is not loaded and there is no corresponding `/dev/video0` device. To load the module (and create the device) you need to do:

    $ sudo modprobe bcm2835-v4l2

V4L is certainly not as bulletproof as `raspivid` and sometimes you may need to reload the module like so:

    $ rmmod bcm2835-v4l2 ; modprobe bcm2835-v4l2

Now let's jump straight into streaming video from the Pi with GStreamer:

    $ v4l2-ctl --vertical_flip=1
    $ gst-launch-1.0 v4l2src ! 'video/x-h264, width=1640, height=922, framerate=42/1' ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=0.0.0.0 port=9090

Then on your local machine:

    $ gst-launch-1.0 tcpclientsrc host=raspberrypi.local port=9090 ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false

Note: starting `gst-launch-1.0` with `-v` can often be useful in getting some insight into the cap filter streams that you need to set, e.g. `video/x-h264, ...` above.

Setting the video format with v4l2-ctl
--------------------------------------

Above it's GStreamer that sets the width, height etc. Just as it's possible to set the camera's vertical flip settings it's also possible to set these other values via `v4l2-ctl`:

    $ v4l2-ctl --set-parm=42
    $ v4l2-ctl --set-fmt-video=width=1640,height=922,pixelformat=H264

`parm` means FPS in V4L's rather strange vernacular. The `pixelformat` value needs to be one of the values displayed by:

    $ v4l2-ctl --list-formats

However this is all a bit academic as GStreamer ignores these values and sets its own. If you don't set width, height and framerate then it defaults to 320x200 at 90fps.

Why it overrides these values but respects e.g. vertical flip, I don't know.

Further v4l2-ctl usage
----------------------

Then you can display all supported video formats including frame sizes:

    $ v4l2-ctl --list-formats-ext

As you can see, on the Pi at least, it's not terribly useful - for each format it just lists one value - the sensor's pixel dimensions.

Then for a format and dimensions you can list the supported FPS range:

    $ v4l2-ctl --list-frameintervals=width=2592,height=1944,pixelformat=H264

Again the output isn't terribly useful - it just seems to list `1-90 fps` for everything. You're better off working with the values shown in the Picamera [documentation](https://picamera.readthedocs.io/en/release-1.12/fov.html#camera-modes) for the various sensor modes.

OK - now onto more useful output. You can see the current V4L settings like so:

    $ v4l2-ctl --all

You can see e.g. the range of values and the current value for auto-exposure:

    auto_exposure (menu)   : min=0 max=3 default=0 value=0

Some settings are marked `(int)` meaning they just take a simple integer value, e.g. brightness takes any value from 0 to 100.

For the ones that are marked `(menu)`, like auto-exposure above, you can see what the numeric values, in this case 0 to 3, mean like so:

    $ v4l2-ctl --list-ctrls-menus

This enumerates the menus for all `(menu)` settings, so for auto-exposure you see that 0 is 'Auto Mode' and 1 is 'Manual Mode' (oddly there's no items for the additional acceptable values 2 and 3).

Loading V4L module on startup
-----------------------------

To automatically load the V4L module at startup do:

    $ echo bcm2835-v4l2 | sudo tee /etc/modules-load.d/v4l2.conf

Taken from the [automatic module loading](https://wiki.archlinux.org/index.php/Kernel_module#Automatic_module_loading_with_systemd) section of ArchLinux page on modules.

Video4Linux2 universal control panel
------------------------------------

To see all this data presented by `v4l2-ctl` in a UI you can use the [Video4Linux2 universal control panel](http://manpages.ubuntu.com/manpages/disco/man1/v4l2ucp.1.html).

Make sure to ssh into your Pi with X11 forwarding enabled, i.e. use `-X`:

    $ ssh -X pi@raspberrypi.local
    $ sudo apt install v4l2ucp
    $ v4l2ucp /dev/video0

You'll see the control panel pop up on your local system. However before you get to that point it'll complain, with a long stream of popups, about situations like the one seen up above (when discussing `(menu)`) where there's no menu text for the auto-exposure values 2 and 3 - with each popup saying something like ""Unable to get menu item for Auto Exposure, index=2. Will use unknown".

To be honest I find the text output of `v4l2-ctl` more digestable.

**Update:** after getting used to the camera and knowing what to look for I did find `v4l2ucp` a bit useful. The first important thing to realize is that the layout is messed up. Things are meant to be laid out row-wise into a number of sections:

* User controls
* Codec controls
* Camera contols
* Auto exposure, bias
* ISO sensitivity
* JPEG compression controls

But the layout is messed up - you have to mentally see every Reset button as marking the end of a row. Once you realize this you can scan down the named settings and spot things such as vertical flip that you may have missed trying to read through the text output of `v4l2-ctl`.

References
----------

There's a lot of stale information about the Pi camera out there. But there are some good references:

* The official documentation for the [Pi camera applications](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).
* The Picamera [hardware page](https://picamera.readthedocs.io/en/release-1.12/fov.html).

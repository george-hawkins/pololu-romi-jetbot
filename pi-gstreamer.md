Pull up the top of the connector and push it backwards (it can only be pushed one direction - lets call that back).

Insert the cable so that the shiny metal connectors at its end are facing forward and the other side of the cable is the side facing the top of the connector that you just pulled up and back.

Now push the top of the connector back into place, boot the Pi and run:

    $ sudo raspi-config

Select Interfacing Options, then select the Camera and go thru the obvious steps to enable it.

---

Install GStreamer on the Pi as per <https://gstreamer.freedesktop.org/documentation/installing/on-linux.html>

Oddly for GStreamer there isn't one single downstream package that pulls in all the needed upstream dependencies.

You install a long list of dependencies - on trying to install all of them the Pi tells you some don't exist (`gstreamer1.0-qt5`, `gstreamer1.0-gtk3` and `gstreamer1.0-gl`) so just leave them out and try again.

On Pi:

    $ raspivid -fps 26 -h 450 -w 600 -vf -n -t 0 -b 200000 -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=192.168.0.66 port=5000

On Ubuntu:

    $ gst-launch-1.0 -v tcpclientsrc host=192.168.0.66 port=5000 ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false

Where 192.168.0.66 is the address of the _Pi_ as shown for `wlan0` by `ifconfig`.

The performance is terrible - the commands come from [here](https://platypus-boats.readthedocs.io/en/latest/source/rpi/video/video-streaming-gstreamer.html#getting-gstreamer) and were written for an earlier version of GStreamer.

To find missing elements:

* Some have just been completely renamed, e.g. ffmpegcolorspace to videoconvert - you just have to google for these.
* Most older elements starting with "ff", e.g. ffdec_h264, have been renamed to "av", so ffdec_h264 become avdec_h264
* You can grep for likely elements with like so `gst-inspect-1.0 | fgrep 264`

---

Streaming video, as above, worked fine but by default there was no `/dev/video0` device and `v4l-ctl` complained about its absense. Creating it required doing:

    $ sudo modprobe bcm2835-v4l2

Then you can display all supported video formats including frame sizes:

    $ v4l2-ctl --list-formats-ext

And then list the supported FPS range for a given format and dimensions:

    $ v4l2-ctl --list-frameintervals=width=2592,height=1944,pixelformat=BGR4

---

Various pages note that `gpu_mem` must be at least 32M with 128M recommended - if you look in `/boot/config.txt` you see that 128M is the pre-configured default. So this hardly seems worth mentioning but maybe at one stage the default for `gpu_mem` was lower. TODO: check if `gpu_mem` is maybe updated to 128M from a lower value when setting up the camera with `raspi-config`.

---

Remember that the [Waveshare fisheye camera](https://www.waveshare.com/rpi-camera-g.htm) uses the older OV5647 sensor of the version 1 Raspberry Pi camera.

So it has the modes listed for the OV5647 in Raspberry Pi [documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).

But the Picamera (python library) documentation has a much better [overview](https://picamera.readthedocs.io/en/release-1.12/fov.html) or the modes with pictures showing the FoV of the older OV5647 sensor and the newer IMX219 sensor.

---

`cvlc` is VLC without a UI - however it still tries to connect to DBUS.

VLC sometimes doesn't release the `/dev/video0` device properly, to fix this remove and reload the underlying module:

    $ rmmod bcm2835-v4l2 ; modprobe bcm2835-v4l2

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

Then on the receiving machine you can start VLC, go to Media / Open Network Stream... and enter the URL `http://192.168.0.66:9090` (where the IP address needs to be the address of _your_ Pi).

The video stream takes a few seconds to start playing and then lags behind reality by a similar amount of time.

Totem, the default player on Ubuntu, works just as well - as do presumably a plethora of other Linux media players. Note: totem uses GStreamer under the covers.

Often on killing and restarting `cvlc` I'd see the error:

    [0071a0b8] v4l2 demux error: cannot start streaming: Operation not permitted
    [0071a0b8] v4l2 demux error: not a radio tuner device

The only solution to this seemed to be to reload the `bcm2835-v4l2` module as outlined above.

Extremely frequently I also saw the error:

    [0093ac18] main decoder error: buffer deadlock prevented

There seemed to be no solution to this other than to repeatedly reload the `bcm2835-v4l2` module and restart `cvlc` until eventually it started without this error. Despite implying that something has been prevented (and the situation recovered) this seems to be a serious error - no streaming occurs if this error appears.

Either my hardware or the v4l2 driver seemed to be very flakey when used like this.

---

You can see the current settings that V4L2 can display like so:

    $ v4l2-ctl --all

You can see e.g. the range of values and the current value for auto-exposure:

    auto_exposure (menu)   : min=0 max=3 default=0 value=0

Some settings are marked `(int)` meaning they just take a simple integer value, e.g. brightness takes any value from 0 to 100.

For the ones that are marked `(menu)` like auto-exposure above you can see what the numerica values, in this case 0 to 3, mean like so:

    $ v4l2-ctl --list-ctrls-menus

This enumerates the menus for all `(menu)` settings, so for auto-exposure you see that 0 is 'Auto Mode' and 1 is 'Manual Mode', oddly there's no items for the additional acceptable values 2 and 3.

---

To see all this data presented in a UI you can use the [Video4Linux2 Universal Control Panel](http://manpages.ubuntu.com/manpages/disco/man1/v4l2ucp.1.html).

Make sure to ssh into your Pi with X11 forwarding enabled, i.e. use `-X`:

    $ ssh -X pi@raspberrypi.local
    $ sudo apt install v4l2ucp
    $ v4l2ucp /dev/video0

You'll see the control panel pop up on your local system (assuming there's a local X server). However before you get to that point it'll complain, with a long stream of popups, about situations like the one seen above where there's no menu text for the auto-exposure values 2 and 3 - with each popup saying something like ""Unable to get menu item for Auto Exposure, index=2. Will use unknown".

To be honest I find the text output of `v4l2-ctl` more digestable.

---

Streaming with `raspivid` proved far less error prone than using V4L2 - it could be killed and restarted repeatedly without seeing any of the issues encountered when using `cvlc` and `/dev/video0`.

Pi:

$ raspivid -v -n -t 0 -md 6 -w 640 -h 480 -fps 20 -vf -l -o tcp://0.0.0.0:2222

Options:

* `-v` - verbose.
* `-n` - disable preview window.
* `-t 0` - start capturing immediately.
* `-md 6 -w 640 -h 480` - mode 6 with width and height set to match the sensor mode.
* `-fps 42` - frames-per-second matching the lower range of mode 6.
* `-vf` - vertical flip (if you appear upside-down).
* `-l -o tcp://0.0.0.0:2222` - listen for TCP connections on port 2222.

See the Picamera [FoV page](https://picamera.readthedocs.io/en/release-1.12/fov.html) for the list of modes and diagrams of what this means it terms of FoV (which the otherwise comprehensive Raspberry Pi [camera documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) doesn't have).

Local:

$ cvlc tcp/h264://raspberrypi.local:2222

The lag is extreme - but presumably this can be tuned? For far less lag:

$ mplayer -nolirc -fps 200 -demuxer h264es ffmpeg://tcp://raspberrypi.local:2222

An FPS of 200 sounds extreme but it doesn't seem to mean what is seems. Leaving it at a far higher value than the FPS than the Pi is producing is fine but lower it too far and lag suddenly appears, so one might as well set it higher than the highest possible Pi FPS.

TODO: I'd like to know what's going on here - it I set both sides to 30 FPS I get serious lag, if I set them both to 42 (with mode 6) then all seems fine but I don't gain anything in quality or reduced CPU usage by setting MPlayer's FPS lower - so it seems sticking with 200 is simplest.

Oddly no matter what the setting lag seems to appear and disappear over time even if there's no lag initially.

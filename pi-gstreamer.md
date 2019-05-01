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

Note that the camera requires that `gpu_mem` be at least 128M - which indeed is default value one sees in `/boot/config.txt` - so there's no need to change it. Perhaps this was once not the case and why it comes up on some pages.

---

Remember that the [Waveshare fisheye camera](https://www.waveshare.com/rpi-camera-g.htm) uses the older OV5647 sensor of the version 1 Raspberry Pi camera.

So it has the modes listed for the OV5647 in Raspberry Pi [documentation](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md).

But the Picamera (python library) documentation has a much better [overview](https://picamera.readthedocs.io/en/release-1.12/fov.html) or the modes with pictures showing the FoV of the older OV5647 sensor and the newer IMX219 sensor.

---

`cvlc` is VLC without a UI - however it still tries to connect to DBUS.

VLC sometimes doesn't release the `/dev/video0` device properly, to fix this remove and reload the underlying module:

    $ rmmod bcm2835-v4l2 ; modprobe bcm2835-v4l2

To stream:

    $ cvlc --intf http --http-port 9090 --http-password SuperSecret v4l2:///dev/video0 --v4l2-width 1920 --v4l2-height 1080 --v4l2-chroma h264 --sout '#standard{access=http,mux=ts,dst=:9091}'

This starts a web interface to VLC on port 9090 and streams video on port 9091.

Then on the receiving machine you can start VLC, go to Media / Open Network Stream... and enter the URL `http://192.168.0.66:9091` (where the IP address is the address of the Pi).

There's a lag of a few seconds before the video starts playing (and the video stream is a similar amount of time behind reality).


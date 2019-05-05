GStreamer on the Nano
=====================

Getting started with using GStreamer with the Raspberry Pi camera module V2 with a Raspberry Pi rather than a Jetson Nano is covered in [`pi-camera-notes.md`](pi-camera-notes.md).

---

    $ sudo apt install v4l-utils
    $ v4l2-ctl --list-formats-ext

Nvidia has optimized versions of various GStreamer elements - use them in preference to the equivalent standard elements:

    $ gst-inspect-1.0 | egrep '^nv'
    $ gst-inspect-1.0 | egrep '^omx'

E.g. use `omxh264enc` rather than `x264enc`.

Basic H.264 steaming
--------------------

On the Nano:

    $ gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM), width=1920, height=1080, framerate=30/1' ! nvvidconv ! omxh264enc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=0.0.0.0 port=9090

On your local machine:

    $ gst-launch-1.0 -v tcpclientsrc host=jetsonnano.local port=9090 ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink sync=false

Most useful page so far - the RidgeRun page of using [GStreamer with the Jetson TX1](https://developer.ridgerun.com/wiki/index.php?title=Gstreamer_pipelines_for_Jetson_TX1).


There's a near identical [page](https://developer.ridgerun.com/wiki/index.php?title=Gstreamer_pipelines_for_Jetson_TX2) for the TX2. The two have diverged slightly - the TX1 page seems to have more reference links - the only section that the TX2 page has that the TX1 doesn't is the short piece on using VLC in the [H264 UDP streaming section](https://developer.ridgerun.com/wiki/index.php?title=Gstreamer_pipelines_for_Jetson_TX2#H264_UDP_Streaming).

---

Intro the GStreamer and Jetson by Peter Moran - <http://petermoran.org/csi-cameras-on-tx2/>

---

Introduction to the GStreamer tools - <https://gstreamer.freedesktop.org/documentation/tutorials/basic/gstreamer-tools.html>

When looking at GStreamer examples it was obvious that a pipeline of tools (called elements) were involved but I didn't understand the bits that seem to be just text strings:

    $ gst-launch-1.0 nvcamerasrc ! 'video/x-raw(memory:NVMM), width=1920, height=1080, format=I420, framerate=60/1' ! nvvidconv ! 'video/x-raw(memory:NVMM), format=I420' ! nvoverlaysink -e

I.e. the bits like `'video/x-raw(memory:NVMM), ...'` - these are [caps filters](https://gstreamer.freedesktop.org/documentation/tutorials/basic/gstreamer-tools.html#caps-filters).

According to the documentation you can achieve the same with [named pads](https://gstreamer.freedesktop.org/documentation/tutorials/basic/gstreamer-tools.html#pads).

TODO: see if named pads allow for simpler specification of what you want, i.e. use `gst-inspect-1.0` to find out the named pads supported by e.g. `nvcamerasrc` and use one of these rather than a big long string.

Elements have capabilities - see <https://gstreamer.freedesktop.org/documentation/tutorials/basic/media-formats-and-pad-capabilities.html>

Elements in a pipeline can produce a whole load of outputs and accept a whole load of inputs - to see an element's sink and source cap(abilities) do:

    $ gst-inspect-1.0 videotestsrc
    $ gst-inspect-1.0 videoconvert

The process by which each of the elements decide on the formats to be used is called caps negotiation - negotiation is covered in a [design section](https://gstreamer.freedesktop.org/documentation/design/negotiation.html) and in a [development section](https://gstreamer.freedesktop.org/documentation/plugin-development/advanced/negotiation.html).

Taking this pipeline:

    $ gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink

Each element supports various capabilities with XXX (what's the name for `format`, `width` etc? Properties?) that can take many values.

It would be interesting to know how exactly the above pipeline settles on:

    video/x-raw, width=320, height=240, framerate=30/1, format=YUY2, pixel-aspect-ratio=1/1, interlace-mode=progressive 

You can see the process and what's settled on with `gst-launch-1.0 -v`.

TODO: what is `-v` actually showing - it doesn't look much like negotiation but it looks like more than just a dump of final choices.

You can get some insight into the process with [pipeline graphs](https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html#getting-pipeline-graphs).

Generate a series of graphs:

    $ mkdir dots
    $ export GST_DEBUG_DUMP_DOT_DIR=$PWD/dots
    $ gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink

Kill `gst-launch` and look at the graphs use `xdot`:

    $ sudo apt install xdot
    $ for file in dots/*; do xdot $file; done

Xdot doesn't have much in the way of a user interface - the author also suggests ZGRViewer in the [links section](https://github.com/jrfonseca/xdot.py#links) of the xdot github page, ZGRViewer is a moribund project (unlike xdot which is actively developed) that provides a Java viewer, that still works fine despite no updates since 2015. While _looking_ far fancier it doesn't seem to offer any additional features over xdot when it comes to the pipeline graphs and for this situation is actually less convenient to use.

---

JetsonHacks has a nice simple [github repo](https://github.com/JetsonHacksNano/CSI-Camera) for getting started with a CSI camera on the Nano.

It covers using `v4l2-ctl` to list what the camera is capable of.

TODO: what other useful V4L2 tools are there?

TODO: this repo uses the element `nvarguscamerasrc` while other pages (including Peter Moran) use `nvcamerasrc` - what's the difference? And how's it different to using `v4l2src device=/dev/video0` - you can see `v4l2src` and `nvcamerasrc` being used [here](https://devtalk.nvidia.com/default/topic/1037844/jetson-tx2/capture-raw-video-through-gstreamer-with-csi-cameras/) to achieve the same affect with a Jetson TX2.

---

Peter Moran's page links to the eLinux Jetson/Cameras [wiki page](https://elinux.org/Jetson/Cameras) which keeps track of supported cameras - for CSI cameras for the Nano it's basically just:

* The Raspberry Pi V2 camera module - $25
* The Leopard Imaging [LI-IMX219-MIPI-FF-NANO](https://leopardimaging.com/product/li-imx219-mipi-ff-nano/) - $29
* The e-Con systems [e-CAM30_CUNANO](https://www.e-consystems.com/nvidia-cameras/jetson-nano-cameras/3mp-mipi-camera.asp) - US$79

---

See RidgeRun's very detailed wiki pages on working with GStreamer on Jetson boards:

* [TX1](https://developer.ridgerun.com/wiki/index.php?title=Gstreamer_pipelines_for_Jetson_TX1)
* [TX2](https://developer.ridgerun.com/wiki/index.php?title=Gstreamer_pipelines_for_Jetson_TX2)

These cover many more tools than just GStreamer. Are they essentially identical?

---

Nvidia [multimedia user guide](https://developer.download.nvidia.com/embedded/L4T/r24_Release_v2.0/Docs/L4T_Tegra_X1_Multimedia_User_Guide_Release_24.2.pdf) - how you find the latest version I don't know, googling throws up various versions but no obvious page that links to the _latest_ version.

Maybe you can get there somehow via the [JetPack sub-site](https://developer.nvidia.com/embedded/jetpack), e.g. via a direct link Google isn't picking up or indirectly via a link in a PDF available there. Or via the the [developer zone](https://developer.nvidia.com/embedded-computing).

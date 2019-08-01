Romi Jetbot
===========

* [`romi-jetbot-bom.md`](romi-jetbot-bom.md) - bill-of-materials for my Romi Jetbot.
* [`jetson-nano-notes.md`](jetson-nano-notes.md) - first-time setup notes for the Nano.
* [`jetson-nano-wifi.md`](jetson-nano-wifi.md) - installing the Intel WiFi module on the Nano development board.
* [`jetson-nano-gstreamer.md`](jetson-nano-gstreamer.md) - streaming video from the Nano using GStreamer.
* [`jetson-power-management/README.md`](jetson-power-management/README.md) - waking the Nano after shutdown without disconnecting and reconnecting the USB power connector.
* [`romi-notes.md`](romi-notes.md) - notes on the Romi - in particular the control board, the basic build and using it with the Pi and Nano.
* [`jetson-nano-power.md`](jetson-nano-power.md) - powering the Nano via the barrel jack using a wall power adapter or batteries.
* [Slightly modified `pololu-rpi-slave-arduino-library`](https://github.com/george-hawkins/pololu-rpi-slave-arduino-library) - see `git log` for more details.

The Pi camera module
--------------------

* [`pi-connecting-camera.md`](pi-connecting-camera.md) - connecting the Pi camera module to a Raspberry Pi.
* [`pi-camera-notes.md`](pi-camera-notes.md) - using the Pi camera module with a Raspberry Pi.
* [`pi-vlc-v4l.md`](pi-vlc-v4l.md) - notes on using VLC with the Pi camera module.

Miscellaneous
-------------

* [`pid.md`](pid.md) - notes on better robot control (mainly concerned with PID control of the motors).
* [`donkey-car-chassis.md`](donkey-car-chassis.md) - notes on alternative [Donkey Car](https://www.donkeycar.com/) chassis.

For the Nano and Donkey Car see also this [blog post](https://medium.com/@feicheung2016/getting-started-with-jetson-nano-and-autonomous-donkey-car-d4f25bbd1c83). For alternative approaches see the [Racecar](https://www.jetsonhacks.com/category/robotics/jetson-racecar/), [Racecar/J](https://www.jetsonhacks.com/category/robotics/racecarj/) and [rover](https://www.jetsonhacks.com/category/robotics/jetson-rover/) tags on JetsonHacks.

Jetson tutorials: <https://developer.nvidia.com/embedded/learn/tutorials> (including a series on using OpenCV with Jetson).

Adafruit Blinka support for Jetson: <https://blog.adafruit.com/2019/03/18/adafruit-blinka-support-for-the-nvidia-jetson-series-nvidia-gtc19-nvidiaembedded/>

Hackaday intro: <https://hackaday.com/2019/03/18/hands-on-new-nvidia-jetson-nano-is-more-power-in-a-smaller-form-factor/>

Nvidia Nano related documentation, resources and other links
------------------------------------------------------------

An irritating thing about the Nvidia site is that one can come across outdated versions of PDFs etc. via Google but it's then quite hard to actually find the current version of the document.

On their site, Nvidia usually use fairly readable _unversioned_ links but these redirect to _versioned_ documents somewhere else entirely.

E.g. <https://developer.nvidia.com/embedded/dlc/jetson-nano-datasheet> _currently_ redirects to <https://developer.download.nvidia.com/assets/embedded/secure/jetson/Nano/docs/NVIDIA_JetsonNano_DataSheet_v0.7.pdf>

People then copy these versioned links, i.e. what shows up in their browser location bar, into their blog entries etc. and after a while these quickly go stale.

The best place to find the most current links is at the [Jetson download center](https://developer.nvidia.com/embedded/downloads) and on [this topic](https://devtalk.nvidia.com/default/topic/1048642/links-to-jetson-nano-resources-amp-wiki/) posted by one of the Nvidia administrators to the Jetson Nano forum.

The forum topic is not being actively updated - however even though the link text may mention a version number, _most_ of the links are unversioned. E.g. "JetPack 4.2 Downloads" links to <https://developer.nvidia.com/embedded/jetpack> where you'll actually find the most recent downloads.

On the download center some of the links seem to be versioned and some not - and some things are released in a rather confusing order, e.g. currently (August 1st, 2019) if you search there for "accelerated GStreamer user guide" you'll see that the most resently released version of this document was 28.3.1 - however if you look further down you'll see version 32.1 was released four months earlier. You probably want the 32.1 version. The version numbers for _some_ documents are tied to [Nvidia JetPack versions](https://developer.nvidia.com/embedded/jetpack) - so 28.3.1 is a minor revision of an older JetPack version while 32.1 is the more recent one.

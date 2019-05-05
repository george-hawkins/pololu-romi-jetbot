VLC and V4L on Raspberry Pi
===========================

As noted on another page here using VLC to stream live video on the Raspberry Pi turned out to be a very unsatisfactory experience and incorrectly led me to assume initially that the V4L implementation of the Raspberry Pi was extremely flakey. So I do not recommend using VLC for this purpose, but here are my notes on using it. Note that `cvlc` is VLC without a GUI - i.e. suitable for a setup where you're accessing a headless Raspberry Pi via ssh.

To stream:

    $ cvlc v4l2:///dev/video0 --v4l2-width 640 --v4l2-height 480 --v4l2-chroma h264 --sout '#standard{access=http,mux=ts,dst=:9090}'

Among other output you see:

    [0175f108] vlcpulse audio output error: PulseAudio server connection failure: Connection refused
    [01758840] dbus interface error: Failed to connect to the D-Bus session daemon: Unable to autolaunch a dbus-daemon without a $DISPLAY for X11
    [016fe110] main libvlc error: interface "dbus,none" initialization failed

It's possible to get rid of the PulseAudio audio error with `--aout dummy` and the D-Bus error with `--no-dbus`. So you end up with:

    $ cvlc --aout dummy --no-dbus v4l2:///dev/video0 --v4l2-width 640 --v4l2-height 480 --v4l2-chroma h264 --sout '#standard{access=http,mux=ts,dst=:9090}'
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

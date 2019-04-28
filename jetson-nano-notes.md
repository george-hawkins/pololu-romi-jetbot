On the first boot the Nano _appeared_ to hang while displaying a dialog that says it's "waiting for unattended-upgr to exit".

But I left it for about 30 minutes and it did eventually move on from this point. Others have experienced this (see this [thread](https://devtalk.nvidia.com/default/topic/1049751/jetson-nano/hangs-at-first-boot-at-quot-waiting-for-unattended-upgr-to-exit-quot-/)) but the solution seems to be simply to wait.

Despite this apparent unattended upgrade process as part of starting up the system still seemed quite stale - doing the following upgraded a substantial number of packages:

    $ sudo apt update
    $ sudo apt full-upgrade
    $ sudo apt autoremove

/etc/ld.so.conf.d/aarch64-linux-gnu_GL.conf: No such file or directory
/etc/ld.so.conf.d/aarch64-linux-gnu_EGL.conf: No such file or directory

    $ cd /etc/ld.so.conf.d
    $ cat aarch64-linux-gnu_GL.conf
    $ cat aarch64-linux-gnu_EGL.conf

    $ ls -l aarch64-linux-gnu_GL.conf
    /etc/alternatives/aarch64-linux-gnu_gl_conf

    $ FILE=$(readlink -f aarch64-linux-gnu_GL.conf)
    $ sudo rm aarch64-linux-gnu_GL.conf
    $ sudo cp $FILE aarch64-linux-gnu_GL.conf

    $ FILE=$(readlink -f aarch64-linux-gnu_EGL.conf)
    $ sudo rm aarch64-linux-gnu_EGL.conf
    $ sudo cp $FILE aarch64-linux-gnu_EGL.conf

    $ sudo update-initramfs -c -k all

Warning: couldn't identify filesystem type for fsck hook, ignoring.

    $ cat /etc/fstab
    $ ls /dev/root
    ls: cannot access '/dev/root': No such file or directory
    $ sudo vim /usr/share/initramfs-tools/scripts/functions

Find the following two lines:

    esac
    [ -e "$DEV" ] && echo "$DEV"

And immediately before them add in the following:

    /dev/root)
        DEV="$(findmnt -n -o SOURCE /)"
        ;;

https://bootlin.com/blog/find-root-device/

    $ sudo update-initramfs -c -k all

    $ fgrep nvrm /var/log/syslog

This may be resolved by adding a camera.

The system doesn't seem particularly responsive but installing and running `htop` doesn't show anything unexpected.

The screen handling seems surprisingly poor given that the system is using nVidia's proprietary drivers and given that nVidia definitely knows how to do HDMI in products like the nVidia shield.

I had to turn off overscan on my TV otherwise the edges of the desktop were offscrean. Overscan seems to be the default on some older TVs but really it should never be on - see this [thread](https://devtalk.nvidia.com/default/topic/1027349/jetson-tx2/display-does-not-fit-properly/). On my Samsung TV disabling overscan involves setting it the picture to "screen fit" - select Menu on the remote control, then select Picture, go down and select Screen Adjustment, then Picture Size and change it to Screen Fit.

---

See [`jetson-nano-wifi.md`](jetson-nano-wifi.md) then...

Once you've plugged back in the keyboad, mouse and HDMI connector - but not the ethernet cable - and reconnected power you'll see you have no internet connection.

Just go Networks (upper right, beside volume etc. in the system menu bar) and select your WiFi network.

----

By default the sound goes to the built-in analog output - it's not clear where this goes as the development board has no audio jack. Perhaps it goes out as I2S on the GPIO header - in which case you'd need something like Adafruit's [I2S breakout](https://www.adafruit.com/product/3006). How audio-in is handled is another matter. Perhaps looking at how things are done with the Snips [dev kit](https://docs.snips.ai/the-maker-kit/dev-kit) would provide some pointers - it uses a [ReSpeaker 2-Mics Pi HAT](http://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/) that features two microphones and a connector for a speaker. Note that this HAT also features various other fairly random extras - two grove connectors, RGBs and a push button.

To get audio out via HDMI Settings / Sound and just select "HDMI / DisplayPort" rather than "Analog Output".

---

Chrome's tendency to gobble all available resources quickly causes the Jetson Nano to grind to a halt - so probably best to keep open tabs to a minimum.

---

Under `/media/$USER/L4T-README` you'll find various interesting files about Linux for Tegra. It's not clear what's mounting this virtual drive. In there you'll find instructions about things like USB dev mode - like the Intel Edison, it looks like you can connect the board via USB to your main system and then run ethernet over USB and then run VNC on the Jetson Nano and interact with it that way.

Other Jetson specific stuff can be found under `/opt/nvidia`.

---

It would be nice to use the Jetson Nano via VNC however this seems largely broken.

It you go to Settings / Desktop Sharing it just crashes.

If you look in `/var/log/syslog` you'll see the reason is that the key `enabled` is missing for `org.gnome.Vino`.

Vino is installed - you can find it under `/usr/lib/vino` and you can see what keys it does support with:

    $ gsettings list-recursively org.gnome.Vino

The issue is covered in Ubuntu [bug 1741027](https://bugs.launchpad.net/ubuntu/+source/unity-control-center/+bug/1741027/) - they claim to have fixed this in November 2018 but it still seems to be present.

This post nVidia [forum post](https://devtalk.nvidia.com/default/topic/1042018/jetson-agx-xavier/setting-up-vnc-server-solved-/) points to the only solution I got to work.

    $ sudo vim /usr/share/glib-2.0/schemas/org.gnome.Vino.gschema.xml

Add the enabled key as described [here](https://bugs.launchpad.net/ubuntu/+source/unity-control-center/+bug/1741027/comments/11).

    $ sudo glib-compile-schemas /usr/share/glib-2.0/schemas

Now you can open Desktop Sharing and turn on sharing - you will need to specify a password if you want to access the system with the standard Mac VNC viewer (as it doesn't support access without a password).

So that's the settings panel fixed - but for whatever reason Vino still doesn't start automatically. You have to start it yourself with:

    $ /usr/lib/vino/vino-server

If you then connect from a remote host everything will work fine but you'll see in the Vino output that it complains about not being able to access `org.freedesktop.Notifications`.

To fix this you have to:

    $ sudo bash
    $ cat > /usr/share/dbus-1/services/org.freedesktop.Notifications.service
    [D-BUS Service]
    Name=org.freedesktop.Notifications
    Exec=/usr/lib/notification-daemon/notification-daemon
    ^D

Getting the vino-server to start automatically on reboot proved impossible - this [SO answer](https://unix.stackexchange.com/a/390919) and other sources suggested:

    $ nmcli c
    $ gsettings set org.gnome.settings-daemon.plugins.sharing.service:/org/gnome/settings-daemon/plugins/sharing/vino-server/ enabled-connections "['183dc28d-fca6-4a9b-a4d5-3cbee6c368a3']"

Where the UUID is taken from the output of `nmcli c`. However this didn't work.

Neither did setting up a systemd service as outlined in this [AskUbuntu answer](https://askubuntu.com/a/1010860). On restarting the status showed:

    $ systemctl enable vinostartup.service
    vinostartup.service - Vino server
       Active: failed (Result: exit-code) since Sun 2019-04-28 21:39:24 CEST; 1min 26s ago

    Apr 28 21:39:24 JetsonNano vino-server[4500]: Unable to init server: Could not connect: Connection refused
    Apr 28 21:39:24 JetsonNano vino-server[4500]: Cannot open display:

But maybe `After` just needs to be set such that the display is actually available by the point this service is run.

In the end I just ssh-ed in to the device:

    $ ssh my-username@JetsonNano.local
    $ export DISPLAY=:0
    $ /usr/lib/vino/vino-server

However the Jetson Nano has to actually be connected to a screen via HDMI otherwise it just creates a tiny low resolution screen.

On my Ubuntu box I used `remote-viewer` to connect to `vnc://JetsonNano.local:5900` - it works but the performance is so poor as to make the whole things rather pointless.

---

Perhaps [x11vnc](https://help.ubuntu.com/community/VNC/Servers#x11vnc) or one of the other VNC servers that don't share a real display would be the way to go. See also this [trouble shooting section](https://wiki.archlinux.org/index.php/X11vnc#Troubleshooting) on using x11vnc with Gnome 3.

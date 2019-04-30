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

---

Preventing the graphical interface from starting just requires setting the default target to boot into to `multi-user.target`. You can see the current target, see the chain of units and set the target to `multi-user.target` like so:

    $ systemctl get-default
    graphical.target
    $ systemd-analyze critical-chain
    graphical.target @3.744s
    `-multi-user.target @3.742s
      `-...
    $ sudo systemctl set-default multi-user.target

Note that many answers on how to disabling the graphical interface also suggest you change `GRUB_CMDLINE_LINUX_DEFAULT` to `text`. The Jetson Nano doesn't override any grub defaults, so you have to create `/etc/default/grub` if you want to do so:

    $ sudo bash
    $ cat > /etc/default/grub
    GRUB_CMDLINE_LINUX_DEFAULT="text"
    ^D
    $ exit

You need to run `update-grub` now - however this isn't installed by default, so you have to do:

    $ sudo apt install grub2-common
    $ sudo update-grub

It turns out this changes nothing, i.e. the non-overriden default is `text` so this step is unnecessary.

By default the NetworkManager won't start WiFi until someone has logged in - either via the graphical interface or via the console. To get it to start WiFi straight away without waiting for someone to log in you need to:

    $ cd /etc/NetworkManager/system-connections 
    $ ls
    $ sudo vim MyWifiSsid

Where `MyWifiSsid` is the file shown by `ls` that has the same name as your WiFi network. Now find the line:

    permissions=user:my-user-name:;

And remove everything _after_ the `=`. See this [SO answer](https://askubuntu.com/a/796962/734204) for more details (and what to do if you have `psk-flags=1` rather than `permissions=...`).

---

Memory usage with graphical envionment:

    $ free
                  total        used        free      shared  buff/cache   available
    Mem:        4051520      907384     2699524       21300      444612     2974532
    Swap:             0           0           0
    $ free -h
                  total        used        free      shared  buff/cache   available
    Mem:           3.9G        1.0G        2.4G         21M        501M        2.7G
    Swap:            0B          0B          0B

Surprisingly the thing with the biggest RSS is `gnome-software` with just 144M - which isn't too shocking.

Memory usage without graphical envionment:

    $ free
                  total        used        free      shared  buff/cache   available
    Mem:        4051520      335504     3466308       17608      249708     3549096
    Swap:             0           0           0
    $ free -h
                  total        used        free      shared  buff/cache   available
    Mem:           3.9G        325M        3.3G         17M        243M        3.4G
    Swap:            0B          0B          0B

Looking at the `available` column you can see the difference is 561M of your 3956M of total memory.

And now the biggest thing running is the `nvargus-daemon` with a RSS of just 19M.

---

Rather than viewing the real display remotely with the default VNC server, which performed terribly, I found using TigerVNC with its own virtual display was infinitely better for remote graphical access to the Jetson Nano.

Install and run TigerVNC:

    $ sudo apt install tigervnc-standalone-server
    $ vncserver -localhost no

The first time you run it, it asks you for a password to use when connecting to it - which it hashes and stores in `~/.vnc/passwd`. It doesn't seem possible not to select a password.

You can kill the server with:

    $ vncserver -kill :1

On your remote machine you can connect with:

    $ xtigervncviewer JetsonNano.local:1

On connecting you'll find a less Unity like, i.e. more vanilla, version of the Gnome desktop - I didn't investigate why it doesn't run with the same configuration as the "real" desktop envionment.

When running the viewer you can press F8 to bring up a context menu that allows you to go full screen etc.

On Ubuntu 18.04 and later you can install the viewer with just:

    $ sudo apt install tigervnc-viewer

But on my old 16.04 system I had to go to <https://bintray.com/tigervnc/stable/tigervnc#files> and then into the `ubuntu-16.04LTS/amd64` subdirectory and download the latest `xtigervncviewer_X.Y.Z-1ubuntu1_amd64.deb`

Then to install it:

    $ cd ~/Downloads
    $ mv xtigervncviewer_*.deb /tmp
    $ cd /tmp
    $ sudo apt install ./xtigervncviewer_*.deb

Moving to `/tmp` just resolves a minor privileges issue when it comes to accessing the directory containing the `.deb`.

Back on the server side you can switch to a more lightweight desktop like so:

    $ vncserver -kill :1
    $ sudo apt-get install lxde
    $ cd ~/.vnc
    cat > xstartup << EOF
    #!/bin/sh
    exec startlxde
    EOF
    $ chmod a+x xstartup
    $ vncserver -localhost no

Before trying [LXDE](https://lxde.org/) I tried [Xfce](https://www.xfce.org/) but Chromium and various other things failed to start (despite following these Arch Linux [instructions](https://wiki.archlinux.org/index.php/TigerVNC#Edit_the_environment_file)). After some investigation I gave up, without coming to any conclusions, and switched to LXDE where things worked without issue.

When running Gnome with TigerVNC, i.e. just connecting without starting anything extra, the memory usage was about the same as when booting into the real graphical envionment:

    $ free
                  total        used        free      shared  buff/cache   available
    Mem:        4059712      983868     2497136       28860      578708     2891508
    Swap:             0           0           0

The `available` amount is slightly more but not really significantly more.

When running LXDE with TigerVNC the memory usage was noticeably lower:

    $ free
                  total        used        free      shared  buff/cache   available
    Mem:        4059712      504564     3075228       29124      479920     3374164
    Swap:             0           0           0

This is a 471M saving on the Gnome setup - so it seems LXDE really does use substantially less resource - at least as far as getting the basic environment up and ready is concerned.

Copy & paste
------------

When starting into the Gnome it automatically starts `vncconfig` which needs to be running for copy & paste to work between the local and remote systems.

With LXDE I had to open a terminal and start `vncconfig` manually - copy & paste worked as expected once this was done.

You can set `vncconfig` to be started automatically by going to the LXDE main menu (lower left) then Preferences / Default applications for LXSession and then to the Autostart tab, entering `@vncconfig -nowin` and hitting the Add button. Note: this just adds an entry to `~/.config/lxsession/LXDE/autostart`. Leaving out the `@`, as I did initially, caused LXPanel to crash (the `@` is supposed to be optional and just marks that you want the application in question to be restarted if it crashes, according to the [documentation](https://wiki.lxde.org/en/LXSession#autostart_configuration_file).

Control of network connections
------------------------------

Both the Gnome and LXDE, when running within TigerVNC, complained that "System policy prevents control of network connections".

To change this:

    $ sudo cp /var/lib/polkit-1/localauthority/10-vendor.d/org.freedesktop.NetworkManager.pkla /var/lib/polkit-1/localauthority/50-local.d
    $ sudo /var/lib/polkit-1/localauthority/50-local.d/org.freedesktop.NetworkManager.pkla

And change `org.freedesktop.NetworkManager.settings.modify.system` to `org.freedesktop.NetworkManager.network-control`. Note: the original `org.freedesktop.NetworkManager.pkla`, that's copied, is just being used as a template for enabling what we need, i.e. `network-control`. See this [blog entry](https://lauri.xn--vsandi-pxa.com/cfgmgmt/polkit.html) on Polkit for more details.

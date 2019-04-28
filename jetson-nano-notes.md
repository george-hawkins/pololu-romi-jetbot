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

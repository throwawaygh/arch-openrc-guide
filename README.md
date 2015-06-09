# How to Install OpenRC on Arch Linux

Whether you appreciate the superior system administration capabilities of
OpenRC, want to reduce boot time, or simply hate systemd, there are plenty of
reasons to want to change your init system. This guide will walk you through
doing it in what I have found to be the most reliable way.


### Note on AUR Packages and Yaourt

You need to install a bunch of AUR packages to get OpenRC working on Arch. For
various reasons, many of them can't be installed with yaourt. I'll note when a
package can be installed with yaourt. I recommend using yaourt whenever
possible, as its automatic dependency handling is really convenient. If you're
unfamiliar with yaourt, look at <https://wiki.archlinux.org/index.php/Yaourt>.

For the sake of completeness, instructions to install a package from the AUR can
be found here:
<https://wiki.archlinux.org/index.php?title=Arch_User_Repository>.


## Installing OpenRC

### Installing SysVinit

The first step is to install SysVinit. I know what you're thinking: isn't this a
guide about installing OpenRC, a different init system? Well, OpenRC runs on top
of SysVinit. In this configuration, all SysVinit does is launch OpenRC and then
take PID 1. This way, the init service manager and PID 1 are separated, so if
your init system goes down, the system can still stay alive.

To install SysVinit, just install the AUR package <a
href="https://aur.archlinux.org/packages/sysvinit/">sysvinit</a>. This can be
done with yaourt.


### Installing OpenRC

To install OpenRC, get the package <a
href="https://aur.archlinux.org/packages/openrc-core/">openrc-core</a>. This can
be done using yaourt. There are several other OpenRC packages in the AUR. Those
will not work with this method.


### Building the Init Scripts

There are three package bases containing init scripts for OpenRC. I recommend
having all three built and installing the packages they contain as needed. The
package bases are <a
href="https://aur.archlinux.org/pkgbase/openrc-base/">openrc-base</a>, <a
href="https://aur.archlinux.org/pkgbase/openrc-desktop/">openrc-desktop</a>, and
<a href="https://aur.archlinux.org/pkgbase/openrc-misc/">openrc-misc</a>.

At the time of writing, openrc-base and openrc-misc have broken PKGBUILD files.
Patched versions are included in this repository. To use the patched version,
replace the PKGBUILD file in the package base with the one from this repository
before running makepkg.

To build a package base, download and extract its archive, replace the PKGBUILD
file if necessary, and then run makepkg from the directory of the PKGBUILD. This
should build several packages in that directory. You should install the packages
for any init services you want. For instance, if you want to use ntp, then
install the ntp-openrc package from the openrc-misc base.

When you install a package, it will tell you how to enable it.


### Enabling Services

To enable a service in OpenRC, use

    # rc-update add <service-name> <runlevel>

Where runlevel is the name of the runlevel, _not_ it's number. For our purposes,
we'll be using the runlevel default.

You can start and stop services using

    # rc-service <service-name> [<command>]

Where command is something like start, stop, restart, etc. Run it with no
command to see the list of available commands.

To see all enabled services, do

    $ rc-update show [<runlevel>]

Finally, to see all of the services you have available, do

    $ ls /etc/init.d

Check out the man pages and the internet for more information.


#### System Logging

You need to have a logging daemon installed. If you don't already have one, I
recommend syslog-ng from the official Arch repositories. It has a service
written for it available in openrc-base. Enable it with

    # rc-update add syslog-ng default

#### cron

You probably want a cron daemon. cronie is a good choice that is available from
the official repositories and has its service available in the openrc-base
package base. Enable it with

    # rc-update add cronie default

#### ALSA

If you want sound, you should add the ALSA service. It is available in the
package alsa-utils-openrc from the openrc-desktop package base. After it's
installed, enable it with

    # rc-update add alsasound default

#### ACPI

Install acpid-openrc from openrc-misc, and run

    # rc-update add acpid default


## Network

Ah, network configuration, the most fun part of setting up any init system. If
you use DHCP, then you should enable the dhcpcd service, which is available in
openrc-base. There are also services for NetworkManager and wicd if you want to
use either of those.

To set up static routing, you need to edit the /etc/conf.d/net file. The file is
well-documented and should tell you everything you need to know. After editing
this file, to enable your interface, you need to make a symlink to
/etc/init.d/net.lo and enable that. For example, if you want to enable an
interface named eno1, you would do:

    # ln -s /etc/init.d/net.lo /etc/init.d/net.eno1
    # rc-update add net.eno1 default

There is also a service for wpa-supplicant if you want to automatically connect
to a wireless network, but I recommend using NetworkManager or wicd instead.


### NetworkManager

If you use NetworkManager, you'll need to build and install <a
href="https://aur.archlinux.org/packages/networkmanager-consolekit/">networkmanager-consolekit</a>
from the AUR. You can use yaourt for this. You'll also need consolekit, so look
at the section "Installing consolekit" below.

When you install the networkmanager-openrc package, it will complain that there
is no service named "networkmanager". This is quite alright. Just run

    # rc-update add NetworkManager default

to enable the service.


### Hostname Configuration

Hostname configuration is done through the file /etc/conf.d/hostname. Put this
line into the file:

    HOSTNAME=darkstar

Where "darkstar" is replaced by whatever you want your hostname to be.

### Enabling avahi

You should enable avahi. Install avahi-openrc from openrc-desktop and run

    # rc-update add avahi default


## Getting Things to Work

### Audio

To enable audio, simply add your user to the audio group.

    # usermod <user> -aG audio


### X

Arch uses logind to provide rootless X. Since logind only works when systemd has
PID 1, we need to make some changes so that users can run X. Edit the file
/etc/X11/Xwrapper.config so that it contains line:

    need_root_rights=yes

Confusingly enough, this makes it so users other than root can start X. If
you're curious about how this works, check out the man page for Xorg.wrap(1).

Also be sure to get DRI working if you want accelerated graphics.


### DRI

In order to access DRI devices, a user needs to be a member of the video group.
Add your user like this:

    # usermod <user> -aG video


### Installing consolekit

consolekit is a very important piece of software that was replaced by logind.
However, now that we've nixed systemd, we want consolekit back. Install it with
the <a href="https://aur.archlinux.org/packages/consolekit/">consolekit</a>
package from the AUR.

While you're at it, I also recommend installing <a
href="https://aur.archlinux.org/packages/polkit-consolekit">polkit-consolekit</a>.
As well as installing and enabling the consolekit-openrc service from the
openrc-desktop package base.

Mercifully, this can all be done using yaourt.


### Desktop Environments and Display Managers

DEs and DMs tend to rely on logind, which, as we said before, doesn't work
anymore. For information about getting specific DEs and DMs working, check out
<https://wiki.archlinux.org/index.php/Openrc#Using_OpenRC_with_a_desktop_environment>.

You'll probably need to install <a
href="https://aur.archlinux.org/packages/consolekit/">consolekit</a> from the
AUR.


## Troubleshooting

At this point, you should be able to reboot your system and have it work. If for
some reason, it doesn't, then use your bootloader to add this to your kernel
parameters

    init=/usr/lib/systemd/systemd

This will start the system with systemd as init, allowing you to go in and
examine your system to see what went wrong.

If you have any problems, check out
<https://wiki.manjaro.org/index.php?title=OpenRC,_an_alternative_to_systemd> and
<https://wiki.parabola.nu/OpenRC>. These wiki articles don't apply perfectly to
Arch, but since both Manjaro and Parabola are based on Arch, they should be
quite helpful.


## Removing systemd

Technically, you can skip this step. However, by following through with it, you
can save some disk space and show Poettering that you mean business.

Make sure you test your system by rebooting *before* you remove systemd. If
OpenRC doesn't work, then you'll have to fix your system with a recovery disk,
which is never fun.

With OpenRC and SysVinit installed, systemd's one remaining job is to provide
udev. In order to finally slay Goliath, we need to replace it with something
else. eudev is a udev fork from Gentoo which works nicely with OpenRC.


### Installing eudev

Installing eudev is a pain in the ass because it conflicts with systemd, so you
get some weird dependency issues. In order to do it, first download <a
href="https://aur.archlinux.org/packages/eudev/">eudev</a> and <a
href="https://aur.archlinux.org/packages/eudev-systemdcompat/">eudev-systemdcompat</a>
from the AUR and unpack them.

    $ tar xzf eudev.tar-gz
    $ tar xzf eudev-systemdcompat.tar.gz

Install some dependencies:

    # pacman -S glib2 kbd kmod hwids util-linux gobject-introspection gperf \
      gtk-doc intltool libgcrypt xz

Next, we want to build both packages. We can't install them yet, because of
dependency issues. Since eudev-systemdcompat depends on eudev, we run makepkg
with the -d flag to ignore dependencies.

    $ cd eudev
    $ makepkg
    $ cd ../eudev-systemdcompat
    $ makepkg -d
    $ cd ..

Finally, install both packages at the same time, removing systemd when prompted.

    # pacman -U eudev/eudev*.pkg.tar.xz eudev-systemdcompat/eudev-systemdcompat.*.pkg.tar.xz

Depending on your system, you may also want to install <a
href="https://aur.archlinux.org/packages/libsystemd-standalone/">libsystemd-standalone</a>
to provide some of the other libraries from systemd.

### Important: Network Interface Names

After installing eudev, network interface names may change. This is most
important if you're using a wireless adapter, because the name will change from
something like wlp1s0 to something like wlan0. To see the available network
interfaces, do:

    $ ls /sys/class/net


## Going Further

At this point, you have a fast, OpenRC-based, systemd-free init.
Congratulations! Here are a few more things you can do from here.


### Fixing Common Boot Errors

You may have some weird boot errors about missing configuration files for sysctl
or tmpfiles.d. These are annoying because they slow down the boot process
substantially. You can fix these errors by running:

    # sudo touch /etc/sysctl.conf /usr/lib/tmpfiles.d/tmp.conf /usr/lib/tmpfiles.d/var.conf


### Enable Parallel Init

If you want a blazing fast boot, you can enable parallel initialization for
OpenRC. You can get it by adding this line to the file /etc/rc.conf

    rc_parallel="YES"

*Disclaimer*: This is still an experimental feature and may break your init (but
it probably won't).


### Add More Runlevels

OpenRC is a powerful system administration tool, so take full advantage of it by
adding more runlevels. For instance, you can have single-user recovery modes,
runlevels without networking, one that starts a display manager, and so on.


### Add More Services

Browse through the available services in the three script package bases we
built as a good starting point. It's also easy to write your own services if
you're familiar with bash.


### Further Information

The man pages for OpenRC and its tools are really helpful. Check them out.

Also, there's a whole internet out there. Look at the wiki pages linked in the
Troubleshooting section for some good starting points.


## Feedback

If there are any problems with this guide, please open an issue. Feel free to
rehost it wherever you want; after all, we need to save as many people as
possible from systemd. Finally, if you have any suggestions to clarify or
improve this guide, please open a pull request.

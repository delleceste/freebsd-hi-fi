# freebsd-hi-fi
FreeBSD multimedia hi fi system

# Base installation

Created a bootable USB stick, I installed FreeBSD on an old Intel NUC from 2016, in a free partition found after the linux dedicated partitions.

#### Eduroam 

To connect to the *eduroam* network, I edited */etc/wpa_supplicant.conf*. I had to use the phone in USB tether mode during the *installation* in order to *install the firmware* that was needed for the WiFi *wlan0* interface.

#### Post install messages

EVDEV_SUPPORT enabled in the kernel -> enable evdev 

> sysctl kern.evdev.rcpt_mask=6

# Kernel timeout messages

controller timeout on  REGISTER Dump adma addr int enab... sdhci-pci0-slot0 blah blah at *boot*:

```
# /boot/loader.conf
hw.sdhci.enable_msi=0

# /boot/device.hints   (append these)
hint.sdhci_pci.0.disabled="1"
hint.sdhci_pci.1.disabled="1"
```

# Software

> pkg install vim git automake wget meson ninja sudo cmake

# Install the destkop environment 

> pkg install  plasma6-plasma sddm  xorg konsole  kdeconnect-kde dolphin

> pkg install drm-kmod

> sysrc kld_list+="i915kms"

> kldload i915kms

> sysrc dbus_enable=YES sddm_enable=YES

> service dbus start

> service sddm onestart

And then, after testing

> sysrc sddm_enable=YES


# KDE Settings

Disable power management suspend session, turn off screen. Disable screen locking in Security and Privacy

Disable animations and unwanted effects

*File search* Data to index: *nothing*

# kdeconnect

# Audio

## upmpdcli

> pkg install libmicrohttpd

> wget https://www.lesbonscomptes.com/upmpdcli/downloads/libnpupnp-6.3.0.tar.gz

> wget https://www.lesbonscomptes.com/upmpdcli/downloads/libupnpp-1.0.4.tar.gz

> wget https://www.lesbonscomptes.com/upmpdcli/downloads/upmpdcli-1.9.17.tar.gz

Build and install with meson

### FreeBSD specific files

> git clone https://github.com/delleceste/upmpdcli-freebsd

> cd upmpdcli-freebsd

> cat freebsd/upmpdcli

Check and then copy

> sudo cp freebsd/upmpdcli /usr/local/etc/rc.d

Add to rc.d



### Qobuz authentication

Read the [manual](https://www.lesbonscomptes.com/upmpdcli/pages/upmpdcli-manual.html#UPMPDCLI-MS-STR-QOBUZ)

## clean pulseaudio  / pipewire

Remove the executables from /usr/local/bin or rename in .no as in Linux

## MPD

pkg install musicpd

We used to have a *resampler* option. No more used. 
musicpd.conf has been cleaned. Removed options have been deleted.

### vim

in *.vimrc*  "*:set mouse="

### bashrc

## Autologin

Drop it in a dedicated file so it survives package updates — /usr/local/etc/sddm.conf.d/autologin.conf:

```
[Autologin]
User=giacomo
Session=plasmax11.desktop
Relogin=false
```


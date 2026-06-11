# freebsd-hi-fi

This repo documents the current FreeBSD setup of `bee`: a Plasma/X11 desktop on an Intel Skylake NUC, with eduroam, Firefox video tuning, and an `open-media-drc`-based audio chain.

The repo layout now mirrors the live filesystem:

```text
etc/rc.conf
etc/wpa_supplicant.conf
open-media-drc/config.env
open-media-drc/etc/devd/usb-audio-drc.conf
open-media-drc/etc/rc.d/musicpd
open-media-drc/etc/rc.d/brutefir_drc
open-media-drc/etc/rc.d/drc_usb_audio
open-media-drc/etc/rc.d/upmpdcli
open-media-drc/mpd/musicpd.conf
open-media-drc/upmpdcli/upmpdcli.conf
usr/local/etc/omdrcctrl/commands.conf
usr/local/etc/rc.d/omdrcctrl
home/giacomo/.config/mozilla/firefox/mmtyb27r.default-release/user.js
home/giacomo/.local/bin/*-nodrc
home/giacomo/.local/share/applications/*-nodrc.desktop
```

Secrets were replaced with placeholders.

## Base install

Useful package milestones recovered from `pkg` install timestamps:

- `2026-05-27`: firmware (`gpu-firmware-intel-kmod-skylake`, `wifi-firmware-iwlwifi-kmod-8000`), `xorg`, `drm-kmod`, `plasma6-plasma`, `sddm`, `seatd`, `konsole`, `dolphin`, `kdeconnect-kde`, build tools (`git`, `wget`, `meson`, `ninja`, `automake`, `sudo`), browsers (`firefox`, `chromium`), plus `cmake`, `libmpdclient`, `libmicrohttpd`
- `2026-05-30`: `musicpd`, `kodi`
- `2026-06-03`: `libnpupnp`, `cantata`
- `2026-06-05`: `linux_base-rl9`, Flask runtime packages for `omdrcctrl`
- `2026-06-06`: `sox`, `libva-intel-media-driver`, `libva-utils`
- `2026-06-11`: `c-ares`, `curl`

Not visible in `pkg` because they were installed outside the package database:

- `brutefir` from the `delleceste/brutefir` fork
- `upmpdcli` and `libupnpp` from source
- `omdrcctrl` built from `/home/giacomo/open-media-drc/omdrc-ctrl`

## Video / GL

`/var/log/Xorg.0.log` shows the actual stack in use:

- Xorg tries `intel`, but the module is absent
- Xorg falls back to `modesetting`
- `glamor` starts on `Mesa Intel(R) HD Graphics 520 (SKL GT2)`
- OpenGL context is `4.6`

So the current FreeBSD graphics path is:

```text
i915kms
-> /dev/dri/card0
-> Xorg modesetting
-> Mesa GL / glamor
```

Packages that matter for this Intel card and GL setup:

- `drm-kmod`
- `gpu-firmware-intel-kmod-skylake`
- `mesa-libs`
- `mesa-dri`
- `xorg`

`xf86-video-intel` is not part of the working setup.

For browser-side video acceleration on this box, the relevant extra packages are `libva`, `libva-intel-media-driver`, and `libva-utils`.

Relevant `rc.conf` lines:

```sh
dbus_enable="YES"
seatd_enable="YES"
sddm_enable="YES"
linux_enable="YES"
kld_list="fusefs i915kms"
```

## Firefox video tuning

The active Firefox overrides are in:

- `home/giacomo/.config/mozilla/firefox/mmtyb27r.default-release/user.js`

The settings used for YouTube/video playback are:

- `media.ffmpeg.vaapi.enabled=true`
- `gfx.x11-egl.force-enabled=true`
- `gfx.webrender.all=true`
- `media.av1.enabled=false`

This enables VA-API on Skylake, forces the X11 EGL path Firefox needs on FreeBSD, forces hardware WebRender past the blocklist, and disables AV1 because HD 520 has no AV1 hardware decode.

## Home directory tweaks

Small local customizations currently present in the home directory:

- `.vimrc`: disables mouse mode and enables syntax highlighting
- `.bashrc`: prepends `$HOME/.local/bin` to `PATH` so locally installed helpers and wrapper scripts are found first

No `.xprofile` is present on this machine.

## Networking / eduroam

The box uses:

```sh
wlans_iwm0="wlan0"
ifconfig_wlan0="WPA  DHCP"
ifconfig_wlan0_ipv6="inet6 ifdisabled  -auto_linklocal"
```

`etc/wpa_supplicant.conf` contains the eduroam profile for:

- `ssid="eduroam"`
- `key_mgmt=WPA-EAP`
- `eap=PEAP`
- `phase2="auth=MSCHAPV2"`

During initial installation, USB tethering was needed so the Wi-Fi firmware could be installed first.

## Current `/etc/rc.conf`

The live machine currently enables:

- `sshd`
- `ntpd`
- `dbus`
- `seatd`
- `sddm`
- `musicpd`
- `upmpdcli`
- `drc_usb_audio`
- `omdrcctrl`

Important detail: `brutefir_drc_enable` is intentionally not enabled. DRC startup is now delegated to `drc_usb_audio` and `devd`, so the chain only starts when the USB DAC is actually present.

## `open-media-drc` and the audio chain

`/home/giacomo/open-media-drc` is a repo-based installation. The point is to keep configs, service glue, helper scripts, and filters in the repo and have the machine read from that checkout directly.

Host-specific values come from `open-media-drc/config.env`, then `install.sh` renders the `*.in` templates into the live files used by the services.

Current audio path:

```text
control point
-> upmpdcli
-> MPD
-> direct DAC output
or
-> virtual_oss loopback
-> brutefir
-> OKTO DAC
```

Important live files mirrored here:

- `open-media-drc/config.env`
- `open-media-drc/mpd/musicpd.conf`
- `open-media-drc/upmpdcli/upmpdcli.conf`

`musicpd.conf` defines:

- direct output to `/dev/dsp0`
- `DRC-native` output to `/dev/dsp.play`
- `DRC-resamp` output to `/dev/dsp.play` at `192000:24:2`

`upmpdcli.conf` currently uses:

- `friendlyname = bee`
- `openhome = 1`
- `upnpav = 0`
- `checkcontentformat = 0`
- `mpdhost = localhost`
- `mpdport = 6600`

## FreeBSD services

Relevant files under `/usr/local/etc/rc.d` on the live system:

```text
musicpd      -> /home/giacomo/open-media-drc/etc/rc.d/musicpd
brutefir_drc -> /home/giacomo/open-media-drc/etc/rc.d/brutefir_drc
drc_usb_audio -> /home/giacomo/open-media-drc/etc/rc.d/drc_usb_audio
upmpdcli     -> /home/giacomo/open-media-drc/etc/rc.d/upmpdcli
omdrcctrl    -> regular file
```

How they work:

- `musicpd`: starts MPD with `/home/giacomo/open-media-drc/mpd/musicpd.conf`
- `upmpdcli`: runs as `giacomo` with `/home/giacomo/open-media-drc/upmpdcli/upmpdcli.conf`
- `brutefir_drc`: worker service that runs `drc.sh restore` and `drc.sh off`
- `drc_usb_audio`: real DRC entry point; probes for `/dev/dsp0`, starts `brutefir_drc` when the DAC is present, and is also triggered by `devd`
- `omdrcctrl`: Flask remote control panel started with `daemon(8)`, running as `giacomo`, with `DISPLAY=:0` and `OMDRCCTRL_DRC_DIR=/home/giacomo/open-media-drc`

`omdrcctrl` command mapping is mirrored in:

- `usr/local/etc/omdrcctrl/commands.conf`

That panel controls:

- DRC off
- DRC native rates `44.1`, `48`, `88.2`, `96`, `192`
- DRC `192 kHz +2dB`
- DRC resampling mode
- app launchers
- reboot / poweroff

## Problems hit so far

### MPD / curl CPU spin

Documented in `/home/giacomo/open-media-drc/MPD-CURL-CPU-SPIN-FreeBSD.md`.

Relevant status to keep in mind:

- the bug was diagnosed as a `libcurl` problem, not an MPD problem
- rebuilding with `c-ares` instead of the threaded resolver was a temporary fix
- the issue is considered solved in curl master

### OKTO DAC 44.1 kHz flicker on FreeBSD

Documented in:

- `OKTO-DAC8-FreeBSD-44k1-flicker.md`
- `freebsd-uaudio-patch/README.md`
- `freebsd-uaudio-patch/FreeBSD-uaudio-shared-clock-bug.md`

Summary:

- stock `uaudio(4)` mishandles the OKTO shared UAC2 clock
- the 44.1 kHz family flickers, while the 48 kHz family is stable
- the local workaround is the patched `snd_uaudio.ko` kept in `open-media-drc/freebsd-uaudio-patch`

### Kodi and `virtual_oss`

Documented in `kodi-virtual-oss-patch/README.md`.

Kodi needed a local OSS sink patch so `virtual_oss` userspace devices become visible and selectable.

## BruteFIR note

BruteFIR is currently installed from a fork which re-introduced the OSS backend for FreeBSD after it was removed from official BruteFIR since `v1.1.0`.

The active geometry in `drc.sh` is:

- `GEOMETRY="120.blue"`

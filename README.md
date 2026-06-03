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

> pkg install vim git automake wget meson ninja sudo cmake python

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

## clean pulseaudio  / pipewire

Remove the executables from /usr/local/bin or rename in .no as in Linux

## MPD

pkg install musicpd  musicpc

We used to have a *resampler* option. No more used. 
musicpd.conf has been cleaned. options removed from the recent releases of MPD have been deleted as well.

## upmpdcli

- *upmpdcli* will be installed *system-wide*

- *upmpdcli* will run as user *giacomo*, and the configuration file shall be local under *~/.local/etc*

- *upmpdcli rc.d script* under */usr/local/etc/rc.d*

> pkg install libmicrohttpd

> wget https://www.lesbonscomptes.com/upmpdcli/downloads/libnpupnp-6.3.0.tar.gz

> wget https://www.lesbonscomptes.com/upmpdcli/downloads/libupnpp-1.0.4.tar.gz

> wget https://www.lesbonscomptes.com/upmpdcli/downloads/upmpdcli-1.9.17.tar.gz

Build and install with meson

### FreeBSD specific files

I created a *fork* of *upmpdcli* and added documentation and the *rc.d* script

> cd /home/giacomo/Downloads

> git clone https://github.com/delleceste/upmpdcli-freebsd

> cd upmpdcli-freebsd

> sudo cp freebsd/upmpdcli /usr/local/etc/rc.d

Add *upmpdcli_enable="YES"* to rc.d

```
upmpdcli_enable="YES"
```

If you want to run *upmpdcli* as user *giacomo* instead of a 
dedicated *upmpdcli* user (that would need creating an additional "nologin" user), check that
the *upmpdcli* *rc* script contains:

```
upmpdcli_user="giacomo"
upmpdcli_homedir="/home/giacomo"
```

> sudo service upmpdcli start

> ps aux |grep upmpdcli

```
giacomo    6494   0.0  0.6     105640  23500  -  I    13:09     0:00.28 /usr/local/bin/upmpdcli -c /home/giacomo/.local/etc/upmpdcli.conf
```

To stop the service:

> sudo service upmpdcli stop

```
Stopping upmpdcli.
Waiting for PIDS: 6494.
```

### Qobuz authentication

Read the [manual](https://www.lesbonscomptes.com/upmpdcli/pages/upmpdcli-manual.html#UPMPDCLI-MS-STR-QOBUZ)

Run the service as user from the command line:

> service upmpdcli stop

>  /usr/local/bin/upmpdcli -c /home/giacomo/.local/etc/upmpdcli.conf -l 4

so that you can read the output log while authenticating on a browser. Use the link provided by:

> ./src/mediaserver/cdplugins/qobuz/qobuz-init-oauth.py  -c /home/giacomo/.local/etc/upmpdcli.conf 

After logging in on the browser, among the *upmpdcli log lines*, you should see:

```
CMDTALK: qobuz-app.py: 'Qobuz running'
0$qobuz$: Qobuz login: oauth initialisation not done
CMDTALK: qobuz-app.py: 'trackuri: [{\'cmdtalk:proc\': \'trackuri\', \'query\': \'{\\n\\t"code_autorisation" : "9YEZ8hhA"\\n}\', \'user-agent\': \'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36\', \'path\': \'/qobuz/oauth/\'}]'
0$qobuz$: Qobuz login: oauth initialisation not done
CMDTALK: qobuz-app.py: 'Qobuz: trackuri: OAuth initialisation'
CMDTALK: qobuz-app.py: 'OAuth: got auth_code: 9YEZ8hhA'
0$qobuz$: session: init_oauth: auth_code 9YEZ8hhA
```

Restart upmpdcli

> sudo service upmpdcli restart

> ps aux |grep upmpd

```
/usr/local/bin/upmpdcli -c /home/giacomo/.local/etc/upmpdcli.conf
```


### brutefir

> cd Downloads

> git clone https://github.com/delleceste/brutefir

Relevant files

- [FreeBSD.md](https://github.com/delleceste/brutefir/blob/master/FreeBSD.md)
  
- freebsd/  directory

#### Build and install

> mkdir build

> cd build

> cmake ..

> make

> sudo make install

```
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/brutefir
-- Set non-toolchain portion of runtime path of "/usr/local/bin/brutefir" to ""
-- Installing: /usr/local/lib/brutefir/file.bfio
-- Installing: /usr/local/lib/brutefir/filecb.bfio
-- Installing: /usr/local/lib/brutefir/cli.bflogic
-- Installing: /usr/local/lib/brutefir/eq.bflogic
-- Set non-toolchain portion of runtime path of "/usr/local/lib/brutefir/eq.bflogic" to ""
-- Installing: /usr/local/lib/brutefir/oss.bfio
-- Installing: /usr/local/etc/rc.d/brutefir_loopback
-- Installing: /usr/local/share/brutefir/brutefir.conf
-- Installing: /usr/local/share/brutefir/brutefir_defaults
```

#### Note

The scripts used to start brutefir will use configuration files under the user's home dir (see brutefir-conf chapter)

#### pass through "dummy" config file

Provide a default configuration "pass through", that is referenced by a compulsory line in the brutefir_defaults.conf file:

> cp /usr/local/share/brutefir/brutefir_passthrough.conf /home/giacomo/.config/BruteFIR/

#### brutefir_defaults file

Location: /home/giacomo/.config/BruteFir/brutefir_defaults  (the modern approach, honoring XDG)

> cp  /usr/local/share/brutefir/brutefir.conf   /home/giacomo/.config/BruteFir/brutefir_defaults

> chown giacomo  /home/giacomo/.config/BruteFir/brutefir_defaults


#### audio loopback devices for brutefir

Relevant directory: *freebsd/rc.d* inside the *brutefir* project

As you can see from the [install output above](#Build-and-install), the installation process installed a file named *brutefir_loopback*
under */usr/local/etc/rc.d*.

Enable it, adding

```
brutefir_loopback_enable="YES"
```

to */etc/rc.conf*

### brutefir-conf

> cd /home/giacomo

> mkdir DRC

> cd DRC

> git clone https://github.com/delleceste/brutefir-conf

> cd brutefir-conf

Inspect *drc.sh*:

```
drc_root="/home/giacomo/DRC"
brutefir_conf_dir="brutefir-conf"
conf_file="$drc_root/$brutefir_conf_dir/brutefir-$1.conf"
process_name="brutefir"
```

and later:

```
echo "Starting 'brutefir $conf_file -daemon'..."
brutefir $conf_file -daemon &>/tmp/brutefir.out
```

The variables shall match the *brutefir-conf* location.

The *mpc* shall be available ( *pkg install musicpc* )


#### Starting brutefir

Try to run the script with the *off* option:

> ./drc.sh off

```
brutefir not running
Output 1 (Bryston BDA-2) is enabled
Output 2 (BDA-2-0.05us_buf) is disabled
Output 3 (DAC+DRC) is disabled
Output 4 (Bryston BDA-2 + resampler) is disabled
DRC stopped
```

##### Options

*drc.sh* accept as option the *SUBSTRING* within the *brutefir-SUBSTRING.conf* files in the same
*brutefir-conf* directory. For example

./drc.sh 120.blue+0dB

### vim

in *~/.vimrc*:

```
:set mouse=
:syntax on
```

### bashrc

## Autologin

Drop it in a dedicated file so it survives package updates — /usr/local/etc/sddm.conf.d/autologin.conf:

```
[Autologin]
User=giacomo
Session=plasmax11.desktop
Relogin=false
```


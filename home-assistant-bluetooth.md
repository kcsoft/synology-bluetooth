# Enable full bluetooth support on Synology

**Tested on DSM 7.2**

I did manage to get full Bluetooth support in Home Assistant (So that the Bluetooth integration doesn't show Failed setup anymore).

Basically you need to install D-Bus service and Bluetooth d-bus agent in Synology (natively).
To do that you can:

1. install [Entware](https://github.com/Entware/Entware/wiki/Install-on-Synology-NAS)

2. install these packages (with opkg)
```
bluez-daemon - 5.66-1
bluez-libs - 5.66-1
bluez-utils - 5.66-1
bluez-utils-btmon - 5.66-1
bluez-utils-extra - 5.66-1
dbus - 1.14.10-1
```

3. copy some configs (run in synology ssh):
```
sudo cp /opt/etc/dbus-1/system.d/bluetooth.conf /etc/dbus-1/system.d/
systemctl restart dbus
sudo ln -s /var/run/dbus/system_bus_socket /opt/var/run/dbus/system_bus_socket
```

4. The HA docker should bind mount the d-bus socket. (can't remember the exact path.. `/var/dbus`? )
You can't mount `/var/dbus` if you started your HA with `Container Manager` -> `Container` (you can't bind system paths). You have to use `Project` with a docker compose yaml. Or (as myself) use another docker manager like [`Portainer`](https://www.portainer.io/).

For testing:

- start in one terminal the bluetooth agent (you can set this to start automatically on boot):
```
sudo bluetoothd -E -n -d
```

- now Bluetooth should work under HA.

You can start a second ssh terminal on Synology and type

```
bluetoothctl
```

##### Bonus: Blutooth audio

If you need audio support for bluetooth you need to compile and install Pulseaudio.

You need to compile the package as it isn't compiled by default.

Install the [Entware build environment](https://github.com/Entware/Entware-ng/wiki/Compile-packages-from-sources)
then run:
```
make menuconfig # select <M> for pulseaudio-avahi, Save
make -j$(nproc) package/pulseaudio/compile
```

```
# add pulse user
sudo synouser --add pulse "" "" 0 "" 0x00
sudo synogroup --add pulse pulse #edit /etc/passwd/group to set uid/gid
sudo synouser --rebuild all

sudo cp /opt/etc/dbus-1/system.d/pulseaudio-system.conf /etc/dbus-1/system.d/
sudo systemctl restart dbus
```




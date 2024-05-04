# synology-bluetooth

**Compile bluetooth modules for Synology DSM7.1**

**The guide for DSM7.2 is [here](https://github.com/kcsoft/synology-bluetooth/blob/master/Compile%20bluetooth%20modules%20for%20Synology%20DSM7.2.md)**

_The guide is for geminilake architecture, kernel version 4.4.180+ but the steps are similar for other architectures._

## Steps

### 1. prepare the environment (Ubuntu 20 was used) ([source](https://help.synology.com/developer-guide/getting_started/prepare_environment.html)):

```sh
mkdir -p /toolkit && cd /toolkit
git clone https://github.com/SynologyOpenSource/pkgscripts-ng
apt-get install cifs-utils python python-pip python3 python3-pip # depend on os
cd /toolkit/pkgscripts-ng/
git checkout DSM7.1
sudo ./EnvDeploy -v 7.1 -p geminilake
```
---

### 2. download the kernel source (for geminilake, kernel 4.4.180)

  * Look for your the kernel for your arch here https://archive.synology.com/download/ToolChain/Synology%20NAS%20GPL%20Source/7.0-41890
  * Search for `"linux-4"` or `"linux-3"` packages.

  * For *geminilake* there is a kernel v4: https://global.synologydownload.com/download/ToolChain/Synology%20NAS%20GPL%20Source/7.0-41890/geminilake/linux-4.4.x.txz
---

### 3. unpack to `/toolkit/build_env/ds.geminilake-7.1usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/`

```sh
tar xvf linux-4.4.x.txz -C /toolkit/build_env/ds.geminilake-7.1/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/
```
---

### 4. enter chroot and compile

```sh
sudo chroot toolkit/build_env/ds.geminilake-7.1

make -C /usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/lib/modules/DSM-7.1/build M=/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/linux-4.4.x/net/bluetooth/ -e CONFIG_BT=m modules

make -C /usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/lib/modules/DSM-7.1/build M=/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/linux-4.4.x/drivers/bluetooth/ -e CONFIG_BT_HCIBTUSB=m modules
# ignore warnings
```
---

### 5. copy `bluetooth.ko` and `btusb.ko` to your NAS `/lib/modules/` folder

```sh
# assuming you have transferred the 2 files on the NAS
sudo cp bluetooth.ko btusb.ko /lib/modules/
```
---

### 6. create a startup file on the NAS to load them on boot

```sh
echo -e "#!/bin/sh\ncase \$1 in\n  start)\n    insmod /lib/modules/bluetooth.ko > /dev/null 2>&1\n    insmod /lib/modules/btusb.ko > /dev/null 2>&1\n    ;;\n  stop)\n    exit 0\n    ;;\n  *)\n    exit 1\n    ;;\nesac" | sudo tee /usr/local/etc/rc.d/bluetooth-modules.sh
sudo chmod 755 /usr/local/etc/rc.d/bluetooth-modules.sh

# execute manually this time, will execute automatically on next boot
sudo /usr/local/etc/rc.d/bluetooth-modules.sh start
```

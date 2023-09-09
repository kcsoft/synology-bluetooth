## using Ubuntu 22.04


#### compiling on NAS, using docker container:
- create an ubuntu 22.04 container, name it `ubuntu`
- run in priviledged mode (need for mounting proc), mount some host folder to `/toolkit`
-  ssh into synology, then run `docker exec -it ubuntu bash`

```sh
# install build apps
apt-get update && apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison wget cifs-utils python2 python-pip python3

# install synology build env for DSM 7.2, geminilake arch
cd /toolkit
git clone https://github.com/SynologyOpenSource/pkgscripts-ng
cd pkgscripts-ng
git checkout DSM7.2
./EnvDeploy -v 7.2 -p geminilake

# download and unpack the kernel source v4.4.302 (using official kernel source, Synology hasn't released the kerner sources at this time)
cd /toolkit
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.302.tar.gz
mkdir -p /toolkit/build_env/ds.geminilake-7.2/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src
tar xvf linux-4.4.302.tar.gz -C /toolkit/build_env/ds.geminilake-7.2/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/

# chroot
mount --bind /dev /toolkit/build_env/ds.geminilake-7.2/dev
chroot /toolkit/build_env/ds.geminilake-7.2/

# we are in chroot, compile the modules
make -C /usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/lib/modules/DSM-7.2/build M=/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/linux-4.4.302/net/bluetooth/ -e CONFIG_BT=m modules

make -C /usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/lib/modules/DSM-7.2/build M=/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/linux-4.4.302/drivers/bluetooth/ -e CONFIG_BT_HCIBTUSB=m modules

exit

# copy the 2 files bluetooth.ko btusb.ko to to NAS /lib/modules/
# /toolkit/build_env/ds.geminilake-7.2/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/linux-4.4.302/net/bluetooth/bluetooth.ko
# /toolkit/build_env/ds.geminilake-7.2/usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/usr/src/linux-4.4.302/drivers/bluetooth//btusb.ko
sudo cp bluetooth.ko btusb.ko /lib/modules/

# create a startup file on nas to load them on boot
echo -e "#!/bin/sh\ncase \$1 in\n start)\n insmod /lib/modules/bluetooth.ko > /dev/null 2>&1\n insmod /lib/modules/btusb.ko > /dev/null 2>&1\n ;;\n stop)\n exit 0\n ;;\n *)\n exit 1\n ;;\nesac" | sudo tee /usr/local/etc/rc.d/bluetooth-modules.sh

sudo chmod 755 /usr/local/etc/rc.d/bluetooth-modules.sh

# execute manually this time
sudo /usr/local/etc/rc.d/bluetooth-modules.sh start

# To test if the module is loaded:
lsmod | grep bt

```

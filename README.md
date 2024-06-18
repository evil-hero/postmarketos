# Xiaomi dipper PostmarketOS

## 0. Clone the repositories
```shell
cd ~
git clone https://github.com/evil-hero/postmarketos.git
```

## 1. Install pmbootstrap
```shell
cd ~/postmarketos
git clone --depth=1 https://gitlab.com/postmarketOS/pmbootstrap.git
mkdir -p ~/.local/bin
ln -s "$PWD/pmbootstrap/pmbootstrap.py" ~/.local/bin/pmbootstrap
```

## 2. pmbootstrap init
```shell
pmbootstrap init
```

## 3. Update config
```shell
rm -rf ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/linux-xiaomi-dipper
rm -rf ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/device-xiaomi-dipper
cp -r ~/postmarketos/linux-xiaomi-dipper ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/
cp -r ~/postmarketos/device-xiaomi-dipper ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/
cp -r ~/postmarketos/firmware-xiaomi-dipper ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/
```

## 4. Build linux
### You should fix the warnings generated by kconfig yourself.
```shell
cd ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/linux-xiaomi-dipper
pmbootstrap checksum linux-xiaomi-dipper
pmbootstrap kconfig edit
```
### Build
!!! A very important step, you need to copy this file and grant execution permission when the `~/.local/var/pmbootstrap/chroot_native/home/pmos/build/src/linux-749b0d4ab2a325e11a8aba7105a0f678a8ecf404/arch/arm64/boot/dts/qcom/sdm845-xiaomi-dipper.dtb` file exists.
```shell
pmbootstrap -j8 build linux-xiaomi-dipper
```
Open a new terminal and execute `pmbootstrap log` to view the log.

## 5. Build device
```shell
cd ~/.local/var/pmbootstrap/cache_git/pmaports/device/testing/device-xiaomi-dipper
pmbootstrap checksum firmware-xiaomi-dipper
pmbootstrap checksum device-xiaomi-dipper
pmbootstrap build device-xiaomi-dipper
```

## 6. Install
Now you need to copy or move the `sdm845-xiaomi-dipper.dtb` file you saved to the `~/.local/var/pmbootstrap/chroot_rootfs_xiaomi-dipper/boot/dtbs/qcom` directory. If the directory does not exist, you can execute `pmbootstrap install` first.

### To install to the system partition of an image file:
```shell
pmbootstrap install
```
### Fix error
```shell
pmbootstrap chroot apk fix --rootfs
```

### Or create an Android recovery zip package:
```shell
pmbootstrap install --android-recovery-zip --recovery-install-partition=data
pmbootstrap flasher --method=adb sideload
```
### Export
```shell
pmbootstrap export
cd $(dirname $(readlink /tmp/postmarketOS-export/pmos-*.zip))
adb sideload pmos-xiaomi-dipper.zip
```

## 7. Flash
```shell
pmbootstrap flasher flash_rootfs --partition userdata
```
### If you have a device that works with fastboot, you can boot the kernel now without flashing it:
```shell
pmbootstrap flasher boot
```
### Otherwise, you will need to flash the kernel to the device boot partition:
```shell
pmbootstrap flasher flash_kernel
```

## 8. Now use the USB network for ssh connection
```shell
# Get the network interface name, usually the last one.
ip a
# Activate the network interface.
ip address add dev enp0s20f0u1 172.16.42.2/24
# Now start the ssh connection.
ssh user@172.16.42.1
```

## 9. Now complete your system
### Update
```shell
sudo apk update
sudo apk upgrade
```
### Connect wifi
```shell
sudo nmcli dev wifi connect "Wifi-ssid" password "Wifi-password" ifname wlan0
sudo nmcli device set wlan0 autoconnect yes
```
### Install the ssh server
```shell
sudo apk add openssh
sudo rc-service sshd start
sudo rc-update add sshd
```
### Fix time zone
```shell
sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
or
```shell
sudo setup-timezone
# type: Asia/Shanghai
```

### Install the docker
```shell
sudo apk add docker docker-compose
sudo addgroup $USER docker
sudo rc-update add docker boot
sudo service docker start
```

## 10. Common commands
```shell
# reboot
sudo reboot

# poweroff
sudo poweroff

# install app
sudo apk add app-name

# uninstall app
sudo apk del app-name

# start service
sudo rc-service docker stop

# stop service
sudo rc-service docker start

# Let the service start with the system
sudo rc-update add docker

# Execute command at startup
sudo echo "touch boot /home/ivon/boot" >> /etc/local.d/boot.start
sudo rc-update add local
```

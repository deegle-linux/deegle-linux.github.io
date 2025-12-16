# Deegle Linux

Deegle Linux is a spare-time toy project with the goal to create a small
embedded Linux distribution for the BeageBone Black and the PocketBeagle 2.
Embedded Linux means that the distro will not have a GUI and act as a "mini PC",
but instead will work autonomously using different attached peripherals,
like sensors and LEDs.

## Repositories

The primary repository is [deegle](https://github.com/deegle-linux/deegle),
and you can find a kind of quick-start guide as
[README.md](https://github.com/deegle-linux/deegle/blob/main/README.md).

The repository [meta-deegle](https://github.com/deegle-linux/meta-deegle) is
the Yocto meta-layer containing the configuration of Deegle Linux, and is
included as submodule by [deegle](https://github.com/deegle-linux/deegle).

The repository [deegle_boot](https://github.com/deegle-linux/deegle_boot) is
a playground for exploring the firmware and bootloaders of the boards,
and not direcly used. If you are interessted in learning about TFTP booting
using the BeagleBone Black, take a look at 
[BeagleBone Black](https://github.com/deegle-linux/deegle_boot?tab=readme-ov-file#beaglebone-black).
Network-booting with [PocketBeagle 2](https://github.com/deegle-linux/deegle_boot?tab=readme-ov-file#pocketbeagle-2)
is work in progress.

## Deegle Linux Distribution

The primary distribution is [deegle](meta-deegle/conf/distro/deegle.conf).
This distribution is extended by [ddeegle](meta-deegle/conf/distro/ddeegle.conf) with
debug tuning like password-less login as `root`.

The Deegle Linux makes use of `systemd` as init manager, because of convenience,
and IPK as package format. It supports networkin with the development host through
ethernet using the USB-C connection, and the debug variant `ddeegle` runs an SSH server.
The development host gets automatically and IP address assigned using the `kea` dhcp server.

## Deegle Linux Images

The Deegel Linux provides the [deegle-base-image](meta-deegle/recipes-core/images/deegle-base-image.bb)
as general purpose base Linux image for embedded applications.
An extended image providing [ROS2 Jazzy](https://docs.ros.org/en/jazzy/index.html)
is in preparation as [deegle-ros2-image](meta-deegle/recipes-core/images/deegle-ros2-image.bb)
but not ready to use at the moment.

## Features

This section describes the Deegle Linux feature implementation.

### USB Ethernet

ID: DEEGLE_USB_ETHERNET

The Deegle Linux makes use of the USB gadget ethernet driver to provide
a interface for communicating with a host PC. This is implemented by
loading the `g_ether` driver on startup, using the `KERNEL_MODULE_AUTOLOAD` 
varlable, implemented in [deegle.conf](meta-deegle/conf/distro/deegle.conf).
To configure the device on startup the `systemd-networkd` service is used.
The `systemd_networkd` service is enabled and configured using
[systemd_%.bbappend](meta-deegle/recipes-core/systemd/systemd_%.bbappend).
The file []() configures the `usb0` inteface to use the IP address 
`192.168.7.1`:

```
[Match]
Name=usb0

[Network]
Address=192.168.7.1/24

[DHCPv4]
UseHostname=false
```

Additionally the `kea` dhcp server is added and 
[configured](meta-deegle/recipes-connectivity/kea/kea_%.bbappend)
to proivde an IPv4 address to the host.
This address should be in the range of `192.168.7.10` to `192.168.7.20`.

### GPIO Support

ID: DEEGLE_GPIO_SUPPORT

The Deegle Linux provides libgpiod and python3-gpio for GPIO access.

## Other configuration

### Disalbe dummy ethernet device

ID: DEEGLE_NO_DUMMY_ETHERNET

The kernel config fragment [dummy.cfg](meta-deegle/recipes-kernel/linux/files/dummy.cfg)
disables the `dummy0` ethernet device, which is useless.

### No connection wait

ID: DEEGLE_NO_CONNECT_WAIT

The [systemd configuration](meta-deegle/recipes-core/systemd/systemd_%.bbappend) is modified
so that the `systemd-networkd-wait-online` service does nothing, since the board is offline
and the USB ethernet is only a debug feature.

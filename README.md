# Chrome OS RMA Shim Bootloader

Shimboot is a collection of scripts for patching a Chrome OS RMA shim to serve as a bootloader for a standard Linux distribution. It allows you to boot a full desktop Debian install on a Chromebook, without needing to unenroll it or modify the firmware.

## About:
Chrome OS RMA shims are bootable disk images which are designed to run a variety of diagnostic utilities on Chromebooks, and they'll work even if the device is enterprise enrolled. Unfortunately for Google, there exists a [security flaw](https://sh1mmer.me/) where the root filesystem of the RMA shim is not verified. This lets us replace the rootfs with anything we want, including a full Linux distribution.

Simply replacing the shim's rootfs doesn't work, as it boots in an environment friendly to the RMA shim, not regular Linux distros. To get around this, a separate bootloader is required to transition from the shim environment to the main rootfs. This bootloader then does `pivot_root` to enter the rootfs, where it then starts the init system.

Another problem is encountered at this stage: the Chrome OS kernel will complain about systemd's mounts, and the boot process will hang. A simple workaround is to [apply a patch](https://github.com/ading2210/chromeos-systemd) to systemd, and then it can be recompiled and hosted at a [repo somewhere](https://shimboot.ading.dev/debian/).

After copying all the firmware from the recovery image and shim to the rootfs, we're able to boot to a mostly working XFCE desktop.

### Partition Layout:
1. 1MB dummy stateful partition
2. 32MB Chrome OS kernel
3. 20MB bootloader
4. The rootfs partitions fill the rest of the disk

Note that rootfs partitions have to be named `shimboot_rootfs:<partname>` for the bootloader to recognize them.

## Status:

### What Works:
- Systmed
- X11
- XFCE
- Backlight
- Touchscreen
- 3D acceleration
- Bluetooth
- Wifi (partially)
- Suspend (partially)

### What Doesn't Work:
- Audio
- Zram

### Development Roadmap:
- ~~build the image automatically~~
- ~~boot to a shell~~
- ~~switch_root into an actual rootfs~~
- ~~start X11 in the actual rootfs~~
- ~~ui improvements in the bootloader~~
- ~~load all needed drivers~~
- ~~autostart X11~~
- ~~host repo for patched systemd packages~~
- ~~use debootstrap to install debian~~
- ~~prompt user for hostname and account when creating the rootfs~~
- ~~auto load iwlmvm~~
- get wifi fully working
- host prebuilt images
- ~~write detailed documentation~~

### Long Term Goals:
- get zram to work
- eliminate binwalk dependency
- get audio to work

## Usage:

### Prerequisites:
- A separate Linux PC for the build process (preferably something Debian-based)
- A USB that is at least 8GB in size
- At least 20GB of free disk space
- An x86-based Chromebook

### Instructions:
1. Grab a Chrome OS RMA Shim from somewhere. Most of them have already been leaked and aren't too difficult to find.
2. Download a Chrome OS [recovery image](https://chromiumdash.appspot.com/serving-builds?deviceCategory=ChromeOS) for your board.
3. Clone this repository and cd into it.
4. Run `sudo ./build_rootfs.sh data/rootfs bookworm` to build the base rootfs.
5. Run `sudo ./patch_rootfs.sh path_to_shim path_to_reco data/rootfs` to patch the base rootfs and add any needed drivers.
6. Run `sudo ./build.sh image.bin path_to_shim data/rootfs` to generate a disk image at `image.bin`. 
7. Flash the generated image to a USB drive or SD card.
8. Enable developer mode on your Chromebook. Even if it's enrolled and dev mode is blocked, it'll still work for running shimboot.
9. Plug the USB into your Chromebook and enter recovery mode. It should detect the USB and run the shimboot bootloader.

Note that these instructions are currently incomplete.

## License:
```
ading2210/shimboot: Boot desktop Linux from a Chrome OS RMA shim.
Copyright (C) 2023 ading2210

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
```
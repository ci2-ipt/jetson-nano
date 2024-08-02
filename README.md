# Installing Jetpack on Jetson Nano w/ WSL2

This is currently an unorganized set of notes I took while trying to flash the latest (supported) Jetpack on a Jetson Nano 4GB.

In brief, it requires Ubuntu 18.04 as host + Jetson Nano booted on USB (+ jumper). In addition, the eMMC space is limited so a NVMe disk was added and configured in order to fully install all the required software. In this case, I used W10 has host and a WSL2 Ubuntu 18.04 VM, in addition, there are also a few tricks to deal with USB (host to guest).

Below list of steps taken, still missing the full description for replication.

1. Install and update WSL
2. Select the right distro (check jetpack versions)
3. If you want to use sdkmanage GUI check the .wslconfig for ram and gui support
4. install all the ubuntu packages that are needed
5. install nvidia sdkmanager
6. register and so on

...
![sdkmanager select deepstream](https://github.com/user-attachments/assets/c1f61c44-8dcd-4c0c-9584-8b382824aa67)

## Flash Jetpack (*Unorganized notes*)
Initial source: https://docs.nvidia.com/sdk-manager/wsl-systems/index.html

### first enter recovery mode
https://wiki.seeedstudio.com/reComputer_J1020_A206_Flash_JetPack/#flashing-jetpack-os-via-nvidia-sdk-manager

1) put jumper 3/4

https://docs.nvidia.com/sdk-manager/wsl-systems/index.html
2) enter WSL2 Ubuntu
3) Connect USB (for data transmit) host -> jetson
4) Boot jetson into Recovery mode (connect to power)

### Attach usb
https://forums.developer.nvidia.com/t/tutorial-using-sdkmanager-for-flashing-on-windows-via-wsl2-wslg/225759
5) first install usbipd-win (install on the Win host)
6) on powershell
```powershell
usbipd wsl list
```
Jetson should appear (VID = 0955:...) and DEVICE = APX

7) attach the usb (jetson) to the WSL vms:
**The right command to achieve this in my case:**
```bash
usbipd wsl list
usbipd.exe attach --wsl -a -b <busid> # -a is **KEY**, as BPMP will restart during flash and needs to auto-reattach
```
Source: https://forums.developer.nvidia.com/t/tutorial-using-sdkmanager-for-flashing-on-windows-via-wsl2-wslg/225759
usbipd wsl list
usbipd wsl attach -a -d <device id> # the -a is key, as BPMP will restart during flashing and need to auto-reattach

**what worked best for my specific case:**
usbipd.exe attach --wsl Ubuntu-18.04 -a -b 2-2

### Check if Jetson is connected /available inside the WSL vm via USB:
```bash
lsusb
# Should output the busid and NVidia Corp.
```

### Run sdkmanager (already installed, if not check add link).
if you get `panda@xxx:~$ SDK Manager is already running; please use the running SDK Manager instance.`, kill the process and start again
```bash
pkill sdkmanager
```

Login! IT is easier to login with QRcode link (top-right corner, then copy link instead of scanning QRcode and proceed)

Select your version (should be auto detected, then probably the non devkit)

If you only have the eMMC do not select DeepStream or you will end out of space during install.

### Expand storage if needed
The way to do this is expand storage with an nvme M.2 disk: check [this](https://wiki.seeedstudio.com/reComputer_Jetson_Memory_Expansion/#expansion-via-m2-slot-on-carrier-board-and-ssd)

Follow instructions, you can flash first the OS to emmc, then change storage to m2, then install sdks.

### Flash!
First flash, after that it will ask for ip and so on to install software
shutdown the jetson
take off the jumper
turn on with monitor, mouse, keyboard, ether (same network)
Change keyboard (18.04 = top right menu, system settings, text entry)
check the IP (ip addr)

Get back to the WSL2 sdkmanager
select ethernet, enter ip / user / pwd and wait for install

**Note:** If "pre-config" is selected you can enter username and pwd, in this case the keyboard layout will be "en" and needs to be changed later.


![sdkmanager flashing jetson](https://github.com/user-attachments/assets/a9b8ef88-abeb-48d7-b7cc-511cdc059e77)
![sdk manager installing software](https://github.com/user-attachments/assets/4c43829c-8923-4c9a-8785-5e7c57b2a6e7)


## Some Random ML Demos to try
https://github.com/dusty-nv/jetson-inference

### webcam part
There is a command line tool which you can use to examine camera capabilities named v4l2-ctl. As shown in the video, to install v4l2-ctl install the v4l-utils debian package:

```bash
sudo apt install v4l-utils
```

Here are some useful commands:
```bash
# List attached devices
$ v4l2-ctl --list-devices
# List all info about a given device
$ v4l2-ctl --all -d /dev/videoX
# Where X is the device number. For example:
$ v4l2-ctl --all -d /dev/video0
# List the cameras pixel formats, images sizes, frame rates
$ v4l2-ctl --list-formats-ext -d /dev/videoX
```

### NVIDIA Capture
https://developer.nvidia.com/embedded/learn/tutorials/first-picture-csi-usb-camera
USB Camera

Runtime command line option
```bash
nvgstcapture-1.0 --camsrc=0 --cap-dev-node=<N> (where N is the /dev/videoN Node)
# Press 'j' to Capture one image.
# Press 'q' to exit
```

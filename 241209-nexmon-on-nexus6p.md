# Reference

* [Nexmon github README](https://github.com/seemoo-lab/nexmon/blob/master/README.md)
* [Issue #195 &quot;install nexmon nexus 6p android 8.0.0 and 8.1&quot;](https://github.com/seemoo-lab/nexmon/issues/195)
* [Issue #305 &quot;nexus 6P nexmon installation pre-compiled or instructions.&quot;](https://github.com/seemoo-lab/nexmon/issues/305)
* [Issue #633 &quot;Can&apos;t setup nexmon for Nexus 6P&quot;](https://github.com/seemoo-lab/nexmon/issues/633)


# Files

* [Magisk module for Nexus 6P](https://github.com/user-attachments/files/18062231/nexmon-magisk-nexus6p.zip)
* [Magisk module for Nexus 6P with debug symbols](https://github.com/user-attachments/files/18062476/nexmon-magisk-nexus6p-debug.zip)


# Prerequisite

## Grant root permission

Unlock the bootloader.

Use [TWRP](https://twrp.me/huawei/huaweinexus6p.html) and [Magisk](https://github.com/topjohnwu/Magisk/releases/) and get root permission on your Nexus 6P.


## Install platform-tools

If Android Studio is already installed, you can get platform tools from SDK manager.

If not, download it [here](https://developer.android.com/tools/releases/platform-tools) or [here (China Mainland users)]().


## Backup original firmware from phone

`adb pull /vendor/firmware/fw_bcmdhd.bin .`

Or use  step `Make a firmware backup` from Build.


# Buiid

Environment: Debian 12


## Install dependencies

```
sudo apt-get install -y build-essential \
  git gawk qpdf adb flex bison git xxd zip
```


## Add i386 arch support for dpkg

```basic
$ sudo dpkg --add-architecture i386
$ sudo apt-get update
$ sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386
```


## Clone repo

```basic
$ cd
$ git clone https://github.com/seemoo-lab/nexmon.git
```


## Start build process

```basic
$ cd nexmon/
$ source setup_env.sh
$ make
```


## Build nexmon firmware

That depends your firmware in the phone. For my phone running Oreo (Android 8.1), it's `7_112_300_14_sta`. You can find more information in nexmon's [README.md](https://github.com/seemoo-lab/nexmon/blob/master/README.md#supported-devices).

```basic
$ cd patches/bcm4358/7_112_300_14_sta/nexmon/
$ make
```

If build is successful, you can find `fw_bcmdhd.bin` in the current directory.

```basic
$ ls
BUILD_NUMBER  fw_bcmdhd.bin  gen  include  log  Makefile  obj  patch.ld  src
```


## Make a firmware backup

`$ make backup-firmware`

If success, you can find the original firmware in current directory.


## Install patched firmware

`$ make install-firmware`

Then reboot your phone by `adb reboot` .


## Build&Install tools and utils

### Mod system approach

It's not enough to just install patched firmware to the phone. We also need some utilities for testing.

Thankfully, nexmon repository provides various program, like `aircrack-ng` and `iw` .

At least we need nexutil to control nexmon firmware. So let's build it now.


1. Setup Android NDK for Nexmon

    ```basic
    $ cd ~
    $ mkdir android-ndk && cd android-ndk
    $ wget "https://dl.google.com/android/repository/android-ndk-r11c-linux-x86_64.zip"
    $ unzip android-ndk-r11c-linux-x86_64.zip
    $ cd android-ndk-r11c
    ```

    If you haven't installed wget or unzip yet, use `sudo apt install -y wget unzip` .

    After uncompressed, you should see strucure like this:

    ```basic
    $ ls
    build         ndk-build    ndk-gdb    ndk-which  prebuilt         source.properties  toolchains
    CHANGELOG.md  ndk-depends  ndk-stack  platforms  python-packages  sources
    ```

    Then we move this directory to `/opt` .

    `sudo mv ./../android-ndk-r11c /opt`
2. Write export to `setup_env.sh`

    Edit `~/nexmon/setup_env.sh` and add these lines before `cd "$OLD_PWD"`.

    ```basic
    export ANDROID_NDK=/opt/android-ndk-r11c
    export PATH=$PATH:$ANDROID_NDK
    export NDK_ROOT=/opt/android-ndk-r11c
    ```

    And source it again in next step.
3. Build tools

    ```basic
    $ cd ~/nexmon/
    $ source setup_env.sh
    $ cd utilities/
    $ make
    ```

    And wait for a while.
4. Install tools to phone

    Please ensure your phone is connected to the computer, and just run `make install`.

    ```
    !!! WRONG APPROACH, JUST FOT RECORD !!!
    Explan: If you need to do system-less mod, please refer to the part.

    We use magisk to root our phone, so it's not ideal to modify system partition. So we are gonna put binaries and libs into vendor partition. For this, we need to modify tools' Makefile to change the location where it is installed.
    Edit the `Makefile` what tool you need to install. Replace `/system` to `/vendor`, then run `make install`. Or you can use this
    `find . -type f -name "Makefile" -exec sed -i 's/\/system/\/vendor/g' {} +`
    to replace.
    Then, in the utilities directory, run `make install` to install all the tools.
    ```


### System-less modding approach

If we need to use Magisk Module for nexmon, we need to modify Makefile.

1. Setup Android NDK

    1. Download Android NDK

        ```basic
        $ cd ~
        $ mkdir android-ndk && cd android-ndk
        $ wget "https://dl.google.com/android/repository/android-ndk-r11c-linux-x86_64.zip"
        $ unzip android-ndk-r11c-linux-x86_64.zip
        $ cd android-ndk-r11c
        ```

        If you haven't installed wget or unzip yet, use `sudo apt install -y wget unzip` .

        After uncompressed, you should see strucure like this:

        ```basic
        $ ls
        build         ndk-build    ndk-gdb    ndk-which  prebuilt         source.properties  toolchains
        CHANGELOG.md  ndk-depends  ndk-stack  platforms  python-packages  sources
        ```

        Then we move this directory to `/opt` .

        `sudo mv ./../android-ndk-r11c /opt`
    2. Write export to `setup_env.sh`

        Edit `~/nexmon/setup_env.sh` and add these lines before `cd "$OLD_PWD"`.

        ```basic
        export ANDROID_NDK=/opt/android-ndk-r11c
        export PATH=$PATH:$ANDROID_NDK
        export NDK_ROOT=/opt/android-ndk-r11c
        ```

        And source it again for the next step.
2. Modify build scripts

    1. Copy magisk module files to the build directory

        `cp -R nexmon/patches/bcm4389c1/20_101_36_2/nexmon/nexmon-magisk ~/nexmon/patches/bcm4358/7_112_300_14_sta/nexmon/`
    2. Modify Makefile

        Add build dep for Makefile. Below the `all: nexmon-magisk.zip` line.

        ```
        magisk: nexmon-magisk.zip

        nexmon-magisk/system/vendor/firmware:
        	@printf "\033[0;31m  CREATING\033[0m magisk module structure %s\n" $@
        	$(Q)mkdir -p $@

        nexmon-magisk/system/bin:
        	@printf "\033[0;31m  CREATING\033[0m magisk module structure %s\n" $@
        	$(Q)mkdir -p $@

        nexmon-magisk/system/bin/nexutil: $(NEXMON_ROOT)/utilities/nexutil nexmon-magisk/system/bin
        	@printf "\033[0;31m  BUILDING\033[0m nexutil %s\n" $@
        	$(Q)make APP_ABI=arm64-v8a -C $<
        	$(Q)cp $(NEXMON_ROOT)/utilities/nexutil/libs/arm64-v8a/nexutil $@

        nexmon-magisk.zip: $(RAM_FILE) nexmon-magisk nexmon-magisk/system/bin/nexutil nexmon-magisk/system/vendor/firmware
        	@printf "\033[0;31m  BUILDING\033[0m magisk module %s (details: log/magisk.log)\n" $@
        	$(Q)cp $< nexmon-magisk/system/vendor/firmware/
        	$(Q)cd nexmon-magisk && zip -r -Z deflate ../$@ * 2>&1 > ../log/magisk.log && cd ..

        ```
3. Build the module

    Run `make magisk` .

    After build process, there is a `nexmon-magisk.zip` under the current directory.
4. Install the magisk module

    Install the zip file in the Magisk app.


# Testing

## (Trying to) Testing monitor mode

So the first step is test monitor mode.

Enter shell and grant permission.

```basic
angler:/ $ su
angler:/vendor/firmware # nexutil -m0
__nex_driver_io: error ret=-1 errno=4
```

And we see this:

```basic
angler:/vendor/firmware # dmesg | grep dhd
[ 2522.478773] dhd_bus_devreset: == Power ON ==
[ 2522.521293] dhd_bus_devreset: dhdpcie_bus_clock_start OK
[ 2522.522103] dhdpcie_dongle_attach: PCI_BAR1_WIN = 0
[ 2522.522762] dhdpcie_dongle_attach: BAR1 window val=23 mask=0
[ 2522.523273] dhdpcie_download_code_file: download firmware /vendor/firmware/fw_bcmdhd.bin
[ 2522.523320] _dhdpcie_download_firmware: dongle image file download failed
[ 2522.523328] dhd_bus_start: failed to download firmware /vendor/firmware/fw_bcmdhd.bin
[ 2522.523334] dhd_bus_devreset: dhd_bus_start: -1
[ 2522.523340] dhd_net_bus_devreset: dhd_bus_devreset: -1
[ 2522.523347] dhd_open : wl_android_wifi_on failed (-1)
[ 2522.523361] dhd_prot_ioctl : bus is down. we have nothing to do
[ 2522.523370] dhd_bus_devreset: == Power OFF ==
[ 2522.644136] dhd_bus_devreset:  WLAN OFF Done
[ 2522.644169] dhd_wlan_power Enter: power off
```


According to [Issue](https://github.com/seemoo-lab/nexmon/issues/195#issuecomment-570985026), we should bring up the interface by our own. And it did works.

```basic
angler:/vendor/firmware # ifconfig wlan0 up
angler:/vendor/firmware # dmesg | grep dhd
[ 2772.285727] \x0aDongle Host Driver, version 1.201.31 (r)\x0aCompiled in drivers/net/wireless/bcmdhd on Oct 11 2018 at 19:39:21
[ 2772.285784] dhd_wlan_power Enter: power on
[ 2772.488624] dhd_bus_devreset: == Power ON ==
[ 2772.531038] dhd_bus_devreset: dhdpcie_bus_clock_start OK
[ 2772.531548] dhdpcie_dongle_attach: PCI_BAR1_WIN = 0
[ 2772.532750] dhdpcie_dongle_attach: BAR1 window val=23 mask=0
[ 2772.533424] dhdpcie_download_code_file: download firmware /vendor/firmware/fw_bcmdhd.bin
[ 2772.609495] dhdpcie_bus_write_vars: Download, Upload and compare of NVRAM succeeded.
[ 2772.611027] [<ffffffc0006cb234>] dhd_bus_download_firmware+0x454/0x4c0
[ 2772.611036] [<ffffffc0006831d4>] dhd_bus_start+0xc0/0x2ac
[ 2772.611043] [<ffffffc0006ce300>] dhd_bus_devreset+0x320/0x3f4
[ 2772.611050] [<ffffffc00067d97c>] dhd_net_bus_devreset+0x94/0xdc
[ 2772.611064] [<ffffffc0006834bc>] dhd_open+0xfc/0x240
[ 2772.641578] Failed to open the file logstrs.bin in dhd_init_logstrs_array, /vendor/firmware/logstrs.bin
[ 2772.822494] dhd_bus_start: Initializing 42 flowrings
[ 2772.822829] dhd_bus_cmn_writeshared:
[ 2772.822841] dhd_bus_cmn_writeshared:
[ 2772.822852] dhd_bus_cmn_writeshared:
[ 2772.822862] dhd_bus_cmn_writeshared:
[ 2772.822873] dhd_bus_cmn_writeshared:
[ 2772.822882] dhd_bus_cmn_writeshared:
[ 2772.822891] dhd_bus_cmn_writeshared:
[ 2772.869150] dhd_prot_ioctl: status ret value is -5
[ 2772.871506] dhd_preinit_ioctls lpc fail WL_DOWN : 0, lpc = 1
[ 2772.872725] dhd_prot_ioctl: status ret value is -23
[ 2772.885956] dhd_prot_ioctl: status ret value is -26
[ 2772.921081] dhd_rtt_init : FTM is supported
[ 2772.924374] dhd_bus_devreset: WLAN Power On Done
angler:/vendor/firmware #
```

It's a little troblesome to bring it up everytime by hand, so I made a Magisk module. <span data-type="text" style="color: var(--b3-font-color1);">(TODO)</span>

And now, we can use nexutil for testing.

```basic
$ LD_PRELOAD=/vendor/lib64/libnexmon.so airodump-ng wlan0
CANNOT LINK EXECUTABLE "airodump-ng": "/vendor/lib64/libnexmon.so" is 64-bit instead of 32-bit
```

So the first problem is, we don't know the binary format of program. We can find it out by `file` command.

```basic
angler:/vendor/lib64 # file /vendor/lib/libnexmon.so
/vendor/lib/libnexmon.so: ELF shared object, 32-bit LSB arm, dynamic (/system/bin/linker), stripped
angler:/vendor/lib64 # file /vendor/bin/airodump-ng
/vendor/bin/airodump-ng: ELF shared object, 32-bit LSB arm, dynamic (/system/bin/linker), stripped
```

We need to use the same architecture binary. So we should run:

```basic
angler:/vendor/lib64 # LD_PRELOAD=/vendor/lib/libnexmon.so airodump-ng wlan0
CANNOT LINK EXECUTABLE "sh": "/vendor/lib/libnexmon.so" is 32-bit instead of 64-bit
```

And another problem is, `sh` provided by system is arm. So we should build a aarch64 binary and use it instead.

The aircrack-ng package only contains arm arch... so we need to use some trick.

```basic
pastel_parasol@ame-vm:~/nexmon/utilities/aircrack-ng$ cd local/
pastel_parasol@ame-vm:~/nexmon/utilities/aircrack-ng/local$ ls
armeabi
pastel_parasol@ame-vm:~/nexmon/utilities/aircrack-ng/local$ cd armeabi/
pastel_parasol@ame-vm:~/nexmon/utilities/aircrack-ng/local/armeabi$ ls
airbase-ng   airdecap-ng    aireplay-ng  airolib-ng  airtun-ng   buddy-ng    ivstools  makeivs-ng  packetforge-ng  wesside-ng
aircrack-ng  airdecloak-ng  airodump-ng  airserv-ng  besside-ng  easside-ng  kstats    objs        tkiptun-ng      wpaclean
pastel_parasol@ame-vm:~/nexmon/utilities/aircrack-ng/local/armeabi$
```

And... the phone freezes while running nexmon.

I opened an issue for this.


## Testing monitor mode

So after messing around with it, I could build the Nexmon Magisk Module for Nexus 6P.

Enter shell and grant permission.

```basic
angler:/ $ su
angler:/ # nexutil -m2
angler:/ # 
```

Now, we don't see the problem above. Let's test it by `airodump-ng`.

Before the executable, according to the readme, we need to provide a radiotap for program. We use `libfakeioctl.so` for this.

```
angler:/ # LD_PRELOAD=libfakeioctl.so airodump-ng wlan0
```

And now it works perfectly. Good.

```

 CH -1 ][ Elapsed: 18 s ][ 2024-12-09 21:50                                       
                                                                                                                                                         
 BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                                         
 xx:xx:xx:xx:xx:xx  -29       25      125   30   1  54e. WPA2 CCMP   PSK  
 (Imagine lots of wireless network here)                                                                         
                                                                                                                                                          
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                
                                                                                                                                                          
 (not associated)   xx:xx:xx:xx:xx:xx  -82    0 - 0      0        9                                                                            
angler:/ # 

```

Now we could use this phone for testing. Yay!


# Trobleshooting

## "xxd: command not found" during make

Description

While building

```basic
  EXTRACTING FIRMWARE BLOB
/usr/bin/bash: line 1: xxd: command not found
```

The problem is because we don't have xxd preinstalled, so build failed.

(If you follow my build process, you won't meet that problem.)


Solution

```basic
$ sudo apt install xxd
```


But there is another problem:

```basic
make[3]: Entering directory '/home/pastel_parasol/nexmon/firmwares/bcm43438/7_45_41_26'
make[3]: Nothing to be done for 'all'.
make[3]: Leaving directory '/home/pastel_parasol/nexmon/firmwares/bcm43438/7_45_41_26'
make[2]: Leaving directory '/home/pastel_parasol/nexmon/firmwares/bcm43438'
  EXECUTING MAKE FOR CHIP VERSION bcm43439a0/
make[2]: Entering directory '/home/pastel_parasol/nexmon/firmwares/bcm43439a0'
  EXECUTING MAKE FOR FIRMWARE VERSION 7_95_49_2271bb6/
make[3]: Entering directory '/home/pastel_parasol/nexmon/firmwares/bcm43439a0/7_95_49_2271bb6'
  EXTRACTING UCODE
ERR: ram file empty or unavailable.
make[3]: *** [Makefile:17: ucode.bin] Error 1
make[3]: Leaving directory '/home/pastel_parasol/nexmon/firmwares/bcm43439a0/7_95_49_2271bb6'
make[2]: *** [Makefile:8: 7_95_49_2271bb6/] Error 2
make[2]: Leaving directory '/home/pastel_parasol/nexmon/firmwares/bcm43439a0'
make[1]: *** [Makefile:7: bcm43439a0/] Error 2
make[1]: Leaving directory '/home/pastel_parasol/nexmon/firmwares'
make: *** [Makefile:5: firmwares] Error 2
```

It's because when xxd is missing, some binaries is already built and `make` thought it passed certain step(but not). So when relaunch make process, it's asking for the missing binary.

Solution

```basic
$ cd firmwares/bcm43439a0/7_95_49_2271bb6/
$ make clean
$ make
```

Example output:

```basic
pastel_parasol@ame-vm:~/nexmon/firmwares/bcm43439a0/7_95_49_2271bb6$ make clean
  CLEANING
pastel_parasol@ame-vm:~/nexmon/firmwares/bcm43439a0/7_95_49_2271bb6$ make
  EXTRACTING FIRMWARE BLOB
  EXTRACTING CLM BLOB
  EXTRACTING UCODE
  EXTRACTING TEMPLATERAM
  EXTRACTING FLASHPATCHES
```

Now you can contiune to make by:

```basic
$ cd ~/nexmon
$ make
```

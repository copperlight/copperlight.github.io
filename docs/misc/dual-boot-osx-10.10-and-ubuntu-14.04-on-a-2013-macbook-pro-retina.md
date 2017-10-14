# Dual Boot OSX 10.10 and Ubuntu 14.04 on a 2013 Macbook Pro Retina

<div class="meta">
  <span class="date"><small>2015-06-24</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

## Introduction

Why?  Some recent benchmarks have shown that Ubuntu can out-perform OSX on Macbook hardware.

* <http://www.phoronix.com/scan.php?page=article&item=osx10_ubuntu1410&num=1>
* <http://www.phoronix.com/scan.php?page=article&item=ubuntu_1404_mba2013gl&num=1>

This process was tested on a late 2013 Macbook11,2 A1398 model:

* 2GHz Intel Core i7
* 16GB 1600 MHz DDR3
* Intel Iris Pro 1536MB

You can check your Macbook model from Ubuntu like so:

```
sudo dmidecode --type 1
```

## Issues

Thunderbolt Monitor support is not great.  It will work if the monitor is connected to the system at
boot-up, but it does not support hot-plug.

!!! note
    See <https://blog.jessfraz.com/post/linux-on-mac/> for some new information on this.

    Apparently, kernel 3.17 has support for hotplugging Thunderbolt connections.

    See <http://ubuntuhandbook.org/index.php/2014/11/how-to-upgrade-to-linux-kernel-3-17-4-in-ubuntu-14-10/> for adding 3.17 to Ubuntu 14.

## Installing

1. Prepare Macbook for Ubuntu installation.

    1. Partition hard disk.

        1. Finder > Applications > Utilities > Disk Utility
        1. Macintosh HD > Partition
        1. Add a "Free Space" partition equal to half the drive.
        1. Apply and wait for the file system to shrink (30-60 minutes).

    1. [Download](http://sourceforge.net/projects/refind/files/0.8.4/refind-bin-0.8.4.zip/download)
    a binary zip and install [rEFInd Boot Manager](http://www.rodsbooks.com/refind/).  This will
    allow you to switch between booting OSX 10.10 and Ubuntu 14.04.

            unzip refind-bin-0.8.4.zip
            cd refind-bin-0.8.4
            ./install.sh --alldrivers

    1. [Download Ubuntu](http://www.ubuntu.com/download/desktop) and create a
    [bootable USB stick](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx).

            hdiutil convert -format UDRW -o ~/path/to/target.img ~/path/to/ubuntu.iso
            diskutil list
            # find usb disk
            diskutil unmountDisk /dev/diskN
            sudo dd if=/path/to/downloaded.img of=/dev/rdiskN bs=1m
            diskutil eject /dev/diskN

1. Install Ubuntu.

    1. Reboot, hold down the Option key and choose Ubuntu USB stick.
    1. Grub Boot Menu: Install Ubuntu
    1. Continue with the installation, without networking.
    1. Installation Type > Something Else
    1. This will allow you to create a custom partition configuration and preserve your Mac OSX install.
    1. You should see a partition list like the following:

        Partition | Type | Size
        ----------|------|-----
        /dev/sda1 | efi | 209 MB
        /dev/sda2 | hfs+ | 125441 MB
        /dev/sda3 | hfs+ | 650 MB
        free space | | 124699 MB

    1. Within the free space, create two new partitions:
        1. Swap space, equal to the same size as system RAM.
        1. An Ext4 partition for the root mount point, consuming the remaining space.
    1. Select the new root partition and continue the installation.
    1. Time Zone: Los Angeles
    1. Keyboard Layout: English (US) - English (Macintosh)
    1. Who Are You?  Re-use your existing username and computer name.
    1. Reboot and choose Ubuntu installation from rEFInd menu.

1. Configure Ubuntu Environment.

    1. Start a Terminal

        1. Ubuntu Search > Terminal
        1. Right-click Launcher Icon > Lock to Launcher

    1. Configure [wireless networking](http://askubuntu.com/questions/513425/how-to-get-wifi-working-on-ubuntu-14-04-macbook-pro-retina-15-inch-2014).

        1. Insert USB stick with Ubuntu installation media.
        1. Install dkms and bcmwl-kernel-source packages.

                cd /media/`whoami`/Ubuntu\ 14.04.1\ LTS\ amd64/
                sudo dpgk -i pool/main/d/dkms/dkms_2.2.0.3-1.1ubuntu5_all.deb
                sudo dpgk -i pool/restricted/b/bcmwl/bcmwl-kernel-source_6.30.223.141+bdcom-0ubuntu2_amd64.deb

        1. Create a network manager wakeup script `/etc/pm/sleep.d/99_wakeup` and make it executable.
        This will restore wireless networking when resuming from suspend.  Bugs
        [1299282](https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1299282) and
        [1289884](https://bugs.launchpad.net/ubuntu/+source/bcmwl/+bug/1289884) are tracking this issue.

                #!/bin/bash

                case "$1" in
                  thaw|resume)
                    nmcli nm sleep false
                    ;;
                esac

                exit $?

    1. Install binary drivers for Intel Iris Pro Graphics 5200.

        1. Download the [64-bit Ubuntu installer](https://download.01.org/gfx/ubuntu/14.04/main/pool/main/i/intel-linux-graphics-installer/intel-linux-graphics-installer_1.0.7-0intel1_amd64.deb)
        from <https://01.org/linuxgraphics/>

                sudo dpkg -i intel-linux-graphics-installer_1.0.7-0intel1_amd64.deb
                sudo apt-get install -f

    1. Disable suspend when closing the lid.

            sudo vi /etc/systemd/logind.conf

            HandleLidSwitch=ignore

            sudo restart systemd-logind

    1. Configure mouse behavior.

        1. System Settings > Mouse and Touchpad
        1. Check: Natural Scrolling
        1. Uncheck: Disable while typing
        1. Uncheck: Tap to click

    1. Disable turning the screen off.

        1. System Settings > Brightness & Lock
        1. Turn Screen Off When Inactive For: Never
        1. Lock Screen After: 10 minutes

    1. Add a screensaver.  The default gnome-screensaver is just a black screen.

            sudo apt-get remove gnome-screensaver
            sudo apt-get install xscreensaver xscreensaver-data-extra xscreensaver-gl-extra

1. Install Applications

    1. [Oracle Java 7 and Java 8](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html)

            sudo add-apt-repository ppa:webupd8team/java
            sudo apt-get update
            echo oracle-java7-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections
            sudo apt-get install oracle-java7-installer

            echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections
            sudo apt-get install oracle-java8-installer

            # switching versions
            sudo update-java-alternatives -s java-7-oracle
            sudo update-java-alternatives -s java-8-oracle
            sudo apt-get install oracle-java8-set-default

    1. [Google Chrome](https://www.google.com/chrome/)

        1. As of version 35,
        [Chrome no longer supports the NPAPI plugin](http://askubuntu.com/questions/470594/how-do-i-get-java-plugin-working-on-google-chrome),
        which is required by the Oracle and OpenJDK Java plugins.  If you need to run a Java plugin
        from a web browser, use Firefox instead.

1. Test Java version.

    1. <https://www.java.com/en/download/testjava.jsp>

1. Test WiFi speed.

    1. Download large files.

            curl -O http://releases.ubuntu.com/14.10/ubuntu-14.10-desktop-amd64.iso
            curl -O http://www.wswd.net/testdownloadfiles/1GB.zip

## Uninstalling

1. Reboot into OSX.

1. Uninstall rEFInd.

        diskutil list
        diskutil mount disk0s1
        sudo su -
        cd /Volumes/EFI
        rm -rf BOOT ubuntu

1. Download [GParted Live](http://gparted.org/download.php) and install it on a USB stick.

        hdiutil convert -format UDRW -o ~/path/to/target.img ~/path/to/gparted.iso
        diskutil list
        # find usb disk
        diskutil unmountDisk /dev/diskN
        sudo dd if=/path/to/downloaded.img of=/dev/rdiskN bs=1m
        diskutil eject /dev/diskN

1. Reboot and hold down the option key.
1. Remove the Linux partitions.
1. Reboot into Mac OSX and extend the partition.

    1. [Covert the CoreStorage volume back into HFS+](http://awesometoast.com/yosemite-core-storage-and-partition-woes/).

            diskutil cs list
            diskutil coreStorage revert $VOLUME_UUID

    1. Reboot.
    1. Delete the recovery partition.
    1. Extend the primary partition.
    1. [Recreate the recovery partition](http://www.techrepublic.com/article/how-to-restore-an-os-x-recovery-partition/).
    1. [Convert the HFS+ volume back to CoreStorage](http://blog.fosketts.net/2012/10/03/mac-os-corestorage-convert/).

            diskutil cs convert "Macintosh HD"

    1. Reboot.

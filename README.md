######  OpenWRT on Raspi

Clone Barrier Breaker from github directory
(I could not build the bluez feeds on Chaos Calmer)

cd SomeDirectory
git clone git://git.openwrt.org/14.07/openwrt.git

cd openwrt
make menuconfig
Target --> Broadcom BCM2708 / BCM2835
Leave the rest as default. Target profile should be RaspberryPi
Save and quit
make

Be patient ...

Create the directory
mkdir /opt/openwrt/package/btkbdemu/src

Clone the source files to the new directory
cd /opt/openwrt/package/btkbdemu/src
rene@Sony:/opt/openwrt/package/btkbdemu/src$ git clone https://github.com/rdubois440/btkbdemu.git .
Cloning into '.'...
remote: Counting objects: 12, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 12 (delta 0), reused 9 (delta 0), pack-reused 0
Unpacking objects: 100% (12/12), done.
Checking connectivity... done.



Double check that the files exist, and can be built
rene@Sony:/opt/openwrt/package/btkbdemu/src$ ls
a.out  btkbdemu.c  hid.h  Makefile  README.md  sdp.h  sdp_record.xml  sdptest.py
rene@Sony:/opt/openwrt/package/btkbdemu/src$ make 
cc -Wall   -c -o btkbdemu.o btkbdemu.c
cc -Wall -lbluetooth btkbdemu.o  -o btkbdemu 
rene@Sony:/opt/openwrt/package/btkbdemu/src$ make clean
rm -f btkbdemu  *.o 

Move the special Makefile.openwrt to the parent directory
mv Makefile.openwrt ../Makefile

Check the last line, it should contain references to explicit libraries
$(eval $(call BuildPackage,btkbdemu,+libusb,+libbluetooth))


At this point, the new package should be available in openwrt build
make menuconfig
Select it

Update the feeds. packages bluez-libs and bluez-utils are in the packages feed

rene@Sony:/opt/openwrt$ gvim feeds.conf.default 
keep only the first line
rene@Sony:/opt/openwrt$ cat feeds.conf.default 
src-git packages https://github.com/openwrt/packages.git;for-15.05
#src-git luci https://github.com/openwrt/luci.git;for-15.05
#src-git routing https://github.com/openwrt-routing/packages.git;for-15.05

./scripts/feeds update -a

rene@Sony:/opt/openwrt$ scripts/feeds install bluez-libs
Installing package 'bluez'
Installing package 'python'
Installing package 'sqlite3'
Installing package 'gdbm'
Installing package 'db47'
Installing package 'libxml2'
Installing package 'libffi'
Installing package 'dbus'
Installing package 'expat'
Installing package 'glib2'
Installing package 'attr'
Installing package 'libical'

rene@Sony:/opt/openwrt$ scripts/feeds install bluez-utils
Collecting package info: done


make menuconfig
Enable bluez-libs and bluez-utils
Kernel Modules --> Other Modules. Select kmod-bluetooth 

make menuconfig again, save the config ...
make

Menu Utilities. Select it

Recreate the SDCard
cd /bin/brcm2708

rene@Sony:/opt/openwrt/bin/brcm2708$ ls -l *.img
-rw-r--r-- 1 rene rene 79691776 Aug 17 10:07 openwrt-brcm2708-bcm2708-sdcard-vfat-ext4.img

Notice the small size, 80 Mb
su to root, 


[root@Sony brcm2708]# dd if=openwrt-brcm2708-bcm2708-sdcard-vfat-ext4.img of=/dev/mmcblk0 bs=1M
76+0 records in
76+0 records out
79691776 bytes (80 MB) copied, 2.66479 s, 29.9 MB/s

Check that it boots 


Boot the sd card
Access the Raspi. 
ssh root@xxxxxxxxxx


PAIR THE DEVICES ! ! !

Make sure bluetooth is started and available. pscan and iscan visibility are not required
root@OpenWrt:~/rene# hciconfig hci0
hci0:   Type: USB
        BD Address: 00:0D:88:8E:E8:49 ACL MTU: 192:8 SCO MTU: 64:8
        UP RUNNING 
        RX bytes:2580 acl:32 sco:0 events:80 errors:0
        TX bytes:881 acl:22 sco:0 commands:49 errors:0



btkbdemu -c FC:19:10:FE:DE:9F


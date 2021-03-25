# mkimg.sh #

This shell script creates a distributable image from a Raspberry Pi SD card.

**NOTE**: This script has **not** been used for imaging irreplaceable data.
While this script *should not* be destructive, it modifies the filesystem and
partition table.

It can be run like this:

```
bash mkimg.sh /dev/sda sdcard.img.zip
```

## What does the script do ##

Under the hood the script performs the following operations:

- ensures the first partition is `fat32` and the second partition `ext4`
- fixes any errors on the filesystem
- shrinks the Linux filesystem to its smallest size
- shrinks the Linux partition to the size of the filesystem + small buffer
- creates a compressed image from the given device
- expands the partition and filesystem back to their largest sizes


## Flash SD card with the image ##

### Linux ###

Replace `<device>` with the location of your SD card; e.g. `/dev/sda`:

```
unzip -p sdcard.img.zip | sudo dd bs=1M of=<device>
```


### Mac ###

Replace `<device>` with the location of your SD card; e.g. `/dev/rdisk1`:

```
unzip -p sdcard.img.zip | sudo dd bs=1m of=<device>
```

### Operating on Disk Image ###
To run the script on a disk image file you can set the image up as a block device and run the script on it.
You'll need to find the partion names and might have to alter the script to match.
```
# Read disk image to file
dd of=sdcard.img if=<device> bs=4096 status=progress

# Mount the image as a device
kpartx -a -f -p p -v sdcard.img

# Locate virtual disk image partitions, check that they're infact the ones you think
fdisk -l /dev/loop
fdisk -l /dev/mapper/loopp2

# Run script with the location of the virtual block device (/dev/loop0) and the location of the virtual block device linux partition (/dev/mapper/loop0p2)
bash mkimg.sh /dev/loop0 sdcard.img.zip /dev/mapper/loop0p2

# Unmount image
kpartx -d -p p -v sdcard.img
```

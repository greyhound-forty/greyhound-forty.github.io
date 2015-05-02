---
layout: post
title: "Introduction to LVM: Part One"
date: 2014-08-14
---

This guide shows you how to work with LVM (Logical Volume Management) on Linux. In this guide I am using an Ubuntu 12 VM running in Hyper-V but besides the install instructions the commands should work pretty much the same in every OS. First let's list some of the highlights of LVM:

*   Resize volume groups online by absorbing new physical volumes (PV) or ejecting existing ones.
*   Resize logical volumes (LV) online by concatenating extents onto them or truncating extents from them.
*   Create read-only snapshots of logical volumes (LVM1).
*   Create RAID logical volumes (since recent LVM implementations, such as Red Hat Enterprise Linux v6 ): RAID1, RAID5, RAID6, etc.
*   Stripe whole or parts of logical volumes across multiple PVs, in a fashion similar to RAID 0.
*   Allocate thin-provisioned logical volumes from a pool
*   Move online logical volumes between PVs.

### Example LVM Layout

![LVM Diagram][1]

### Let's Begin

To get started let's look at the current disks that are not being used.

    [telesto] (~) >>> fdisk -l|egrep 'contain a valid partition table'
    Disk /dev/sdb doesn't contain a valid partition table
    Disk /dev/sdc doesn't contain a valid partition table
    Disk /dev/sdd doesn't contain a valid partition table


As you can see there are no partitions yet on /dev/sdb - /dev/sdd. We will go ahead and create the partition for sdb and then you can repeat the steps for sdc and sdd. I have stripped some of the fdisk output for formatting reasons:

    [telesto] (~) >>> fdisk /dev/sdb
    Partition type:
       p   primary (0 primary, 0 extended, 4 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-20971519, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):
    Using default value 20971519
    Command (m for help): t
    Selected partition 1
    Hex code (type L to list codes): 8e
    Changed system type of partition 1 to 8e (Linux LVM)
    Command (m for help): w


Go ahead and repeat these steps for /dev/sdc and /dev/sdd ensuring that you use the 't' command to mark the new partitions as LVM.

### Physical Volumes

Now that are partitions are created let's prepare our new partitions for LVM

    [telesto] (~) >>> pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1
      Physical volume "/dev/sdb1" successfully created
      Physical volume "/dev/sdc1" successfully created
      Physical volume "/dev/sdd1" successfully created


If you need to remove the Physical Volumes you can simply use the pvremove command:

    [telesto] (~) >>> pvremove /dev/sdb1 /dev/sdc1 /dev/sdd1
      Labels on physical volume "/dev/sdb1" successfully wiped
      Labels on physical volume "/dev/sdc1" successfully wiped
      Labels on physical volume "/dev/sdd1" successfully wiped


Now that we have shown how to remove the Physical volumes, let's go ahead and create them one more time:

    [telesto] (~) >>> pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1
      Physical volume "/dev/sdb1" successfully created
      Physical volume "/dev/sdc1" successfully created
      Physical volume "/dev/sdd1" successfully created


We can check on the state of the Physical Volumes by using the pvdisplay command:

    [telesto] (~) >>> pvdisplay
      --- Physical volume ---
      PV Name               /dev/sda5
      VG Name               telesto-vg
      PV Size               19.76 GiB / not usable 2.00 MiB
      Allocatable           yes
      PE Size               4.00 MiB
      Total PE              5058
      Free PE               5
      Allocated PE          5053
      PV UUID               6rm54Y-GFwa-fz5N-xpji-Hnnh-N6Eu-0b58MP

      "/dev/sdb1" is a new physical volume of "10.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/sdb1
      VG Name
      PV Size               10.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               Ea9mWF-dxip-hA3n-AqoK-Q47M-f3IJ-OnLr9Q

      "/dev/sdc1" is a new physical volume of "10.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/sdc1
      VG Name
      PV Size               10.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               BD3fMg-VLsy-rYf9-P6gN-FNdB-lTLL-YemEJU

      "/dev/sdd1" is a new physical volume of "10.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/sdd1
      VG Name
      PV Size               10.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               vambW6-lZyD-1Jgk-TqMi-IAYy-SU4Q-0KpF1Z


### Volume Groups

Now let's create our Volume group. A Volume group is a set of physical partitions that are presented to your operating system as Logical volumes. In this case we are creating a Volume group called 'data' and adding our 3 extra drives to the group

    [telesto] (~) >>> vgcreate data /dev/sdb1 /dev/sdc1 /dev/sdd1
    Volume group "data" successfully created


To get the state of your current Volume groups use the vgdisplay command:

    [telesto] (~) >>> vgdisplay
      --- Volume group ---
      VG Name               data
      System ID
      Format                lvm2
      Metadata Areas        3
      Metadata Sequence No  1
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                0
      Open LV               0
      Max PV                0
      Cur PV                3
      Act PV                3
      VG Size               29.99 GiB
      PE Size               4.00 MiB
      Total PE              7677
      Alloc PE / Size       0 / 0
      Free  PE / Size       7677 / 29.99 GiB
      VG UUID               DWO3xh-8v9i-GA5J-BklW-gDts-Ic5r-7jIrZa

      --- Volume group ---
      VG Name               telesto-vg
      System ID
      Format                lvm2
      Metadata Areas        1
      Metadata Sequence No  3
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                2
      Open LV               2
      Max PV                0
      Cur PV                1
      Act PV                1
      VG Size               19.76 GiB
      PE Size               4.00 MiB
      Total PE              5058
      Alloc PE / Size       5053 / 19.74 GiB
      Free  PE / Size       5 / 20.00 MiB
      VG UUID               6pnQaD-Uoft-mN5y-enmi-kojN-2qH7-AcfPdR


Another important command for working with Volume groups is vgsan. The vgscan command scans all SCSI, IDE, and many other disk devices in the system looking for LVM physical volumes and volume groups.

    [telesto] (~) >>> vgscan
    Reading all physical volumes.  This may take a while...
    Found volume group "data" using metadata type lvm2
    Found volume group "telesto-vg" using metadata type lvm2


Let's go ahead and rename our current Volume group from data to testing and rescan the system with vgscan:

    [telesto] (~) >>> vgrename data testing
    Volume group "data" successfully renamed to "testing"

    [telesto] (~) >>> vgscan
    Reading all physical volumes.  This may take a while...
    Found volume group "testing" using metadata type lvm2
    Found volume group "telesto-vg" using metadata type lvm2

    [telesto] (~) >>> vgdisplay
      --- Volume group ---
      VG Name               testing
      System ID
      Format                lvm2
      Metadata Areas        3
      Metadata Sequence No  2
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                0
      Open LV               0
      Max PV                0
      Cur PV                3
      Act PV                3
      VG Size               29.99 GiB
      PE Size               4.00 MiB
      Total PE              7677
      Alloc PE / Size       0 / 0
      Free  PE / Size       7677 / 29.99 GiB
      VG UUID               DWO3xh-8v9i-GA5J-BklW-gDts-Ic5r-7jIrZa

      --- Volume group ---
      VG Name               telesto-vg
      System ID
      Format                lvm2
      Metadata Areas        1
      Metadata Sequence No  3
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                2
      Open LV               2
      Max PV                0
      Cur PV                1
      Act PV                1
      VG Size               19.76 GiB
      PE Size               4.00 MiB
      Total PE              5058
      Alloc PE / Size       5053 / 19.74 GiB
      Free  PE / Size       5 / 20.00 MiB
      VG UUID               6pnQaD-Uoft-mN5y-enmi-kojN-2qH7-AcfPdR


Like with Physical volumes we can also remove Volume groups. For this we use the `vgremove` command:

    [telesto] (~) >>> vgremove testing
    Volume group "testing" successfully removed
    [telesto] (~) >>> vgscan
    Reading all physical volumes.  This may take a while...
    Found volume group "telesto-vg" using metadata type lvm2


Now we can recreate our Volume group so that we can move on to working with Logical volumes:

    [telesto] (~) >>> vgcreate data /dev/sdb1 /dev/sdc1 /dev/sdd1
    Volume group "data" successfully created  
    [telesto] (~) >>> vgscan
    Reading all physical volumes.  This may take a while...
    Found volume group "data" using metadata type lvm2
    Found volume group "telesto-vg" using metadata type lvm2


### Logical Volumes

Next we move on to creating our logical volumes named backup (5GB), nfs (5GB), and media (2GB) in our Volume group data.

    [telesto] (~) >>> lvcreate --name backup --size 5G data
      Logical volume "backup" created

    [telesto] (~) >>> lvcreate --name nfs --size 5G data
      Logical volume "nfs" created

    [telesto] (~) >>> lvcreate --name media --size 2G data
      Logical volume "media" created


Let's get an overview of our Logical volumes by issuing the `lvdisplay` command:

    [telesto] (~) >>> lvdisplay
      --- Logical volume ---
      LV Name                /dev/data/backup
      VG Name                data
      LV UUID                AOsCPj-SXkq-09Ev-jJU9-pKaZ-JfmS-VDQqln
      LV Write Access        read/write
      LV Status              available
      # open                 0
      LV Size                5.00 GiB
      Current LE             1280
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:2

      --- Logical volume ---
      LV Name                /dev/data/nfs
      VG Name                data
      LV UUID                Q40nWG-i2c9-KS8A-wChV-EmDF-bgJC-8kM12U
      LV Write Access        read/write
      LV Status              available
      # open                 0
      LV Size                5.00 GiB
      Current LE             1280
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:3

      --- Logical volume ---
      LV Name                /dev/data/media
      VG Name                data
      LV UUID                hN0duL-AJu9-OhrI-kstI-pAGI-e8f6-5i0P3P
      LV Write Access        read/write
      LV Status              available
      # open                 0
      LV Size                2.00 GiB
      Current LE             512
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:4

      --- Logical volume ---
      LV Name                /dev/telesto-vg/root
      VG Name                telesto-vg
      LV UUID                JUfWbF-i5tC-mLGE-SVVG-Arwi-CXT8-vLnTxv
      LV Write Access        read/write
      LV Status              available
      # open                 1
      LV Size                17.74 GiB
      Current LE             4542
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:0

      --- Logical volume ---
      LV Name                /dev/telesto-vg/swap_1
      VG Name                telesto-vg
      LV UUID                72mC2a-Vy79-FYB3-Pj7E-excU-wKzY-FhoRMT
      LV Write Access        read/write
      LV Status              available
      # open                 2
      LV Size                2.00 GiB
      Current LE             511
      Segments               1
      Allocation             inherit
      Read ahead sectors     auto
      - currently set to     256
      Block device           252:1


We can issue `lvscan` to look at the system and identify Logical volumes:

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/data/backup' [5.00 GiB] inherit
      ACTIVE            '/dev/data/nfs' [5.00 GiB] inherit
      ACTIVE            '/dev/data/media' [2.00 GiB] inherit
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit


Like with the Volume groups, we can rename Logical Volumes on the fly. In this case we are going to use `lvrename` to rename nfs to exports

    [telesto] (~) >>> lvrename data nfs exports
      Renamed "nfs" to "exports" in volume group "data"

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/data/backup' [5.00 GiB] inherit
      ACTIVE            '/dev/data/exports' [5.00 GiB] inherit
      ACTIVE            '/dev/data/media' [2.00 GiB] inherit
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit


For further practice let's remove the Logical volume exports now using the `lvremove` command. This command actually requires the full path to the Logical volume you would like to remove:

    [telesto] (~) >>> lvremove /dev/data/exports
    Do you really want to remove active logical volume exports? [y/n]: y
      Logical volume "exports" successfully removed

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/data/backup' [5.00 GiB] inherit
      ACTIVE            '/dev/data/media' [2.00 GiB] inherit
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit


Let's create the logical volume nfs again:

    [telesto] (~) >>> lvcreate --name nfs --size 5G data
      Logical volume "nfs" created

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/data/backup' [5.00 GiB] inherit
      ACTIVE            '/dev/data/media' [2.00 GiB] inherit
      ACTIVE            '/dev/data/nfs' [5.00 GiB] inherit
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit


Another big benefit of LVM is the ability to extend and contract storage as needed. In this example we will extend the backup Logical volume by 5GB for a total of 10GB using the `lvextend` command. This command also requires the full path to the Volume group:

    [telesto] (~) >>> lvextend -L10G /dev/data/backup
      Extending logical volume backup to 10.00 GiB
      Logical volume backup successfully resized


To reduce the size of the Volume group you would use the aptly named `lvreduce` command. Let's shrink the backup volume down to 7GB:

    [telesto] (~) >>> lvreduce -L7G /dev/data/backup
      WARNING: Reducing active logical volume to 7.00 GiB
      THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce backup? [y/n]: y
      Reducing logical volume backup to 7.00 GiB
      Logical volume backup successfully resized

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/data/backup' [7.00 GiB] inherit
      ACTIVE            '/dev/data/media' [2.00 GiB] inherit
      ACTIVE            '/dev/data/nfs' [5.00 GiB] inherit
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit


This concludes the first introduction to LVM. In part 2 we will go in to creating the Filesystems and integrating the new LVM bits in to our running system.

### Remove LVM and reset server

Yes if you were smart enough to take a snapshot of your system you could simply roll back, but I thought it would be useful to show how to properly remove your LVM tracks. Let's start by removing the Logical Volumes we created:

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/data/backup' [7.00 GiB] inherit
      ACTIVE            '/dev/data/media' [2.00 GiB] inherit
      ACTIVE            '/dev/data/nfs' [5.00 GiB] inherit
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit

    [telesto] (~) >>> lvremove /dev/data/nfs /dev/data/media /dev/data/backup
    Do you really want to remove active logical volume media? [y/n]: y
      Logical volume "media" successfully removed
    Do you really want to remove active logical volume nfs? [y/n]: y
      Logical volume "nfs" successfully removed
    Do you really want to remove active logical volume backup? [y/n]: y
      Logical volume "backup" successfully removed

    [telesto] (~) >>> lvscan
      ACTIVE            '/dev/telesto-vg/root' [17.74 GiB] inherit
      ACTIVE            '/dev/telesto-vg/swap_1' [2.00 GiB] inherit


The next layer to peel away is the Volume group we created. Let's go ahead and remove that:

    [telesto] (~) >>> vgscan
      Reading all physical volumes.  This may take a while...
      Found volume group "data" using metadata type lvm2
      Found volume group "telesto-vg" using metadata type lvm2
     07:45:37 PM
    [telesto] (~) >>> vgremove data
      Volume group "data" successfully removed

    [telesto] (~) >>> vgscan
      Reading all physical volumes.  This may take a while...
      Found volume group "telesto-vg" using metadata type lvm2


Two more layers to go. This time let's remove the Physical Volume

    [telesto] (~) >>> pvscan
      PV /dev/sda5   VG telesto-vg      lvm2 [19.76 GiB / 20.00 MiB free]
      PV /dev/sdb1                      lvm2 [10.00 GiB]
      PV /dev/sdc1                      lvm2 [10.00 GiB]
      PV /dev/sdd1                      lvm2 [10.00 GiB]
      Total: 4 [49.75 GiB] / in use: 1 [19.76 GiB] / in no VG: 3 [30.00 GiB]

    [telesto] (~) >>> pvremove /dev/sdb1 /dev/sdc1 /dev/sdd1
      Labels on physical volume "/dev/sdb1" successfully wiped
      Labels on physical volume "/dev/sdc1" successfully wiped
      Labels on physical volume "/dev/sdd1" successfully wiped

    [telesto] (~) >>> pvscan
      PV /dev/sda5   VG telesto-vg   lvm2 [19.76 GiB / 20.00 MiB free]
      Total: 1 [19.76 GiB] / in use: 1 [19.76 GiB] / in no VG: 0 [0   ]


And finally we remove the wipe the partition table from the drive using [fdisk][2] or [parted][3]. In this example I only show wiping 2 of the drives. Use one of the mentioned tools to wipe the third drives partition table:

    [telesto] (~) >>> fdisk /dev/sdb  

    Command (m for help): d
    Selected partition 1

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

    [telesto] (~) >>> parted /dev/sdc
    GNU Parted 2.3
    Using /dev/sdc
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) p
    Model: Msft Virtual Disk (scsi)
    Disk /dev/sdc: 10.7GB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos

    Number  Start   End     Size    Type     File system  Flags
     1      1049kB  10.7GB  10.7GB  primary               lvm

    (parted) rm 1
    (parted) quit
    Information: You may need to update /etc/fstab.


To remove your MBR & partition table you would then run the following for each drive. Please type this one out or at least copy/pasta in to a text editor first and make sure you are not accidentally wiping a drive that is in use:

    dd if=/dev/zero of=/dev/sdb bs=512 count=1


This guide was made possible by the following sites:

[A Beginner's Guide To LVM on howtoforge.com][4]  
[Logical Volume Manager Administration by Redhat][5]

 [1]: http://i.imgur.com/IENkhog.png "LVM Image Outline"
 [2]: http://example.com "Fdisk Guide"
 [3]: http://example.com "Parted Guide"
 [4]: http://www.howtoforge.com/linux_lvm "A Beginner's Guide To LVM"
 [5]: https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html-single/Logical_Volume_Manager_Administration/ "Logical Volume Manager Administration"

---
layout: post
title: "Introduction to LVM: Part Two"
date: 2014-08-16
---

If you need a refresher on where we are at this point please see [Part 1][1] of our Introduction to LVM guide.

Now we have three logical volumes, but no actual filesystems so they won't do us much good when trying to store data on them. We'll create an ext3 filesystem on backup, an ext4 filesystem on nfs and a xfs filesystem on media

    [telesto] (~) >>> mkfs.ext3 /dev/data/backup
    mke2fs 1.42 (29-Nov-2011)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    327680 inodes, 1310720 blocks
    65536 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=1342177280
    40 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912, 819200, 884736

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done

    [telesto] (~) >>> mkfs.ext4 /dev/data/nfs
    mke2fs 1.42 (29-Nov-2011)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    327680 inodes, 1310720 blocks
    65536 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=1342177280
    40 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912, 819200, 884736

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done


    [telesto] (~) >>> mkfs.xfs /dev/data/media
    meta-data=/dev/data/media        isize=256    agcount=4, agsize=131072 blks
             =                       sectsz=512   attr=2, projid32bit=0
    data     =                       bsize=4096   blocks=524288, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0


Now we are ready to mount our logical volumes, so let's create the mount points and get them all mounted:

    [telesto] (~) >>> mkdir -p /var/{media,backup,nfs}

    [telesto] (~) >>> mount /dev/data/backup /var/backup

    [telesto] (~) >>> mount /dev/data/nfs /var/nfs

    [telesto] (~) >>> mount /dev/data/media /var/media

    [telesto] (~) >>> mount |grep '/var'
    /dev/mapper/data-backup on /var/backup type ext3 (rw)
    /dev/mapper/data-nfs on /var/nfs type ext4 (rw)
    /dev/mapper/data-media on /var/media type xfs (rw)


Now we have a functional LVM system and we can read and write to the newly mounted partitions. For the volumes to be mounted on boot of course we need to adjust our /etc/fstab file. First we'll make a backup in case anything gets borked and we need to rescue the system from boot up issues:

    [telesto] (~) >>> cp /etc/fstab{,.bak}

    [telesto] (~) >>> ll /etc/fstab*
    -rw-r--r-- 1 root root  767 Apr  1 08:38 /etc/fstab
    -rw-r--r-- 1 root root  767 Apr 10 08:49 /etc/fstab.bak


Now that we have a good backup we need to add the new mounts to the file. Using your favorite editor add the following entries to the /etc/fstab file:

    /dev/data/backup   /var/backup     ext3       rw,noatime    0 0
    /dev/data/nfs    /var/nfs      ext4        rw,noatime    0 0
    /dev/data/media    /var/media      xfs   rw,noatime    0 0


To test that we have everything done correctly we will now reboot. Execute the command `shutdown -r now` to reboot the system.

Once the system is back online we can run `df -h` or `mount` to make sure that everything we added to the fstab file automatically mounted on boot up.

    [telesto] (~) >>> df -h
    Filesystem                    Size  Used Avail Use% Mounted on
    /dev/mapper/telesto--vg-root   18G  1.5G   15G   9% /
    udev                          989M  4.0K  989M   1% /dev
    tmpfs                         200M  276K  200M   1% /run
    none                          5.0M     0  5.0M   0% /run/lock
    none                          998M     0  998M   0% /run/shm
    /dev/mapper/data-media        2.0G   33M  2.0G   2% /var/media
    /dev/sda1                     228M   54M  163M  25% /boot
    /dev/mapper/data-backup       5.0G  139M  4.6G   3% /var/backup
    /dev/mapper/data-nfs          4.8G   10M  4.6G   1% /var/nfs


As I mentioned previously one of the real benefits of LVM is the ability to resize Logical volumes and now we can take that a step further and resize not only the volumes but their respective filesystems as well. The first volume we are going to deal with is our logical volume *backup*.

First we need to umount the volume

    [telesto] (~) >>> umount /var/backup

    [telesto] (~) >>> mount|grep backup


If the umount was successful you should get no outout from the `mount|grep backup` command (provided you have no other mounts with the name backup in them).

Now let's enlarge *backup* from 5GB to 10GB

    [telesto] (~) >>> lvextend -L10GB /dev/data/backup
      Extending logical volume backup to 10.00 GiB
      Logical volume backup successfully resized


This of course only enlarges the logical volume and not the filesystem. To enlarge the filesystem we need to use the commands `e2fsck` and `resize2fs`

    [telesto] (~) >>> e2fsck -f /dev/data/backup
    e2fsck 1.42 (29-Nov-2011)
    Pass 1: Checking inodes, blocks, and sizes
    Pass 2: Checking directory structure
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    /dev/data/backup: 11/327680 files (0.0% non-contiguous), 55935/1310720 blocks


We need to make a note here about the total block amount (1310720) for when we shrink the volume later in this guide.

    [telesto] (~) >>> resize2fs /dev/data/backup
    resize2fs 1.42 (29-Nov-2011)
    Resizing the filesystem on /dev/data/backup to 2621440 (4k) blocks.
    The filesystem on /dev/data/backup is now 2621440 blocks long.


Now that we have successfully enlarged the logical volume *backup* and resized the filesystem we will need to remount it:

    [telesto] (~) >>> mount /dev/data/backup /var/backup

    [telesto] (~) >>> df -h
    Filesystem                    Size  Used Avail Use% Mounted on
    /dev/mapper/telesto--vg-root   18G  1.5G   15G   9% /
    udev                          989M  4.0K  989M   1% /dev
    tmpfs                         200M  276K  200M   1% /run
    none                          5.0M     0  5.0M   0% /run/lock
    none                          998M     0  998M   0% /run/shm
    /dev/mapper/data-media        2.0G   33M  2.0G   2% /var/media
    /dev/sda1                     228M   54M  163M  25% /boot
    /dev/mapper/data-nfs          4.8G   10M  4.6G   1% /var/nfs
    /dev/mapper/data-backup       9.9G  140M  9.3G   2% /var/backup


If we need to shrink a logical volume we actually need to work in the reverse order. The first step is to shrink the filesystem and then we can actually reduve the logical volume itself. Let's pick on the *backup* volume one more time. First step as always is to umount it:

    [telesto] (~) >>> umount /var/backup

    [telesto] (~) >>> df -h
    Filesystem                    Size  Used Avail Use% Mounted on
    /dev/mapper/telesto--vg-root   18G  1.5G   15G   9% /
    udev                          989M  4.0K  989M   1% /dev
    tmpfs                         200M  276K  200M   1% /run
    none                          5.0M     0  5.0M   0% /run/lock
    none                          998M     0  998M   0% /run/shm
    /dev/mapper/data-media        2.0G   33M  2.0G   2% /var/media
    /dev/sda1                     228M   54M  163M  25% /boot
    /dev/mapper/data-nfs          4.8G   10M  4.6G   1% /var/nfs


This is where we need that block number we saved from earlier. When you are resizing an ext3 filesystem to a specific size (instead of all available space) you need to specify the number of blocks. You can specify the size in MB,GB, etc but we already saves that block number so we'll use that.

    [telesto] (~) >>> resize2fs /dev/data/backup 1310720
    resize2fs 1.42 (29-Nov-2011)
    Resizing the filesystem on /dev/data/backup to 1310720 (4k) blocks.
    The filesystem on /dev/data/backup is now 1310720 blocks long.


With our filesystem size now reduced we need to use `lvreduce` to shrink the logical volume as well. Note that this is a destructive process and you will be prompted to acknowledge this fact. Since we have not put any data on this filesystem we can safely ignore the warning

    [telesto] (~) >>> lvreduce -L5G /dev/data/backup
      WARNING: Reducing active logical volume to 5.00 GiB
      THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce backup? [y/n]: y
      Reducing logical volume backup to 5.00 GiB
      Logical volume backup successfully resized


With all that done we can now remount the logical volume

    [telesto] (~) >>> mount /dev/data/backup /var/backup

    [telesto] (~) >>> df -h
    Filesystem                    Size  Used Avail Use% Mounted on
    /dev/mapper/telesto--vg-root   18G  1.5G   15G   9% /
    udev                          989M  4.0K  989M   1% /dev
    tmpfs                         200M  276K  200M   1% /run
    none                          5.0M     0  5.0M   0% /run/lock
    none                          998M     0  998M   0% /run/shm
    /dev/mapper/data-media        2.0G   33M  2.0G   2% /var/media
    /dev/sda1                     228M   54M  163M  25% /boot
    /dev/mapper/data-nfs          4.8G   10M  4.6G   1% /var/nfs
    /dev/mapper/data-backup       5.0G  139M  4.6G   3% /var/backup


That completes part 2 of our Introduction to LVM series. In the next part of the series we will learn how to setup LVM and software Raid.

 [1]: http://inchhighassassins.com/?page_id=24

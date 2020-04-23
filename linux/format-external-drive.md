# How to quickly partition and format an external drive

## Find the drive to partition

`lsblk`

The command should show a list of disks. Identify the drive to partition, e.g. `/dev/sdb`

## Partition the drive

Use *gpt* (https://www.howtogeek.com/193669/whats-the-difference-between-gpt-and-mbr-when-partitioning-a-drive/)

```
sudo parted /dev/sdb mklabel gpt

sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
```

The drive is partitioned using the `ext4` filesystem.

Run `lsblk` to verify that the partition is correctly created.

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdb     8:0    0   100G  0 disk 
└─sdb1  8:1    0   100G  0 part
```

## Create the filesystem

```
sudo mkfs.ext4 -L datapartition /dev/sdb1
```

The label `datapartition` can be changed at any time with the `e2label` command.

## Mount the filesystem

Create the mount-point

```
sudo mkdir -p /mnt/data
```

Check the `uid` of the partition: `sudo lsblk --fs`

Add this line to `/etc/fstab`

```
UUID=[uid] /mnt/data ext4 defaults 0 2
```

Finally, mount the drive:

`sudo mount -a`

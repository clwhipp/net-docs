# RAID

Data stored within the network is stored within a series of RAID
arrays. RAID, or Redundant Array of Inexpensive Disks, is a technology
where multiple physical disks are utilized to prevent data loss in event
of disk failure.

There are several modes or types of RAID that can be utilized with varying
levels of advantages and disadvantages. The RAIDs in place as of now are
configured in a RAID-1 (or mirrored) configuration. This means that any data
written to the RAID are written across all the physical disks in the RAID.

RAIDs can be implemented within Hardware or Software. There are various
trade-offs that come into play when considerig whether to utilize hardware
or software.

| Criteria | Software | Hardware |
|----------|----------|----------|
| Performance | Slower performance as CPU utilized. | Higher performance as dedicated hardware used. |
| Complexity of Administration | Increased complexity as admin responsible for all aspects such as creation, monitoring, and disk management. | Simpler as admin only responsible for enable data writes to drives. |
| Vendor Lock-in | No Vendor Lock | Common for hardware controllers to utilize different formats that are incompatible. |
| Alerting Capabilities | Scripting and automation can be developed to detect failed disks and notify admins. | Hardware controllers responsible for detection and alerting restricted to what is enabled by hardware. |
| Flexibility | Most flexible as RAIDs are configured and managed within software. As such, custom setups are different combinations of commands. | No flexibility beyond what is configured within hardware controller firmware. |
| Support bug fixes and updates | Very supportive as it's just a matter of updating a software package. | Not nearly as flexible as it's a hardware firmware updates. Dependent upon hardware developers maintaining their product lines for full length of time that it's present in your environment. |
| Security | Support software updates to fix bugs. | Security updates may never happen depending upon controllers ability to reload. |

The trade-offs documented in the above table was considered when
choosing whether to utilize software or hardware based RAID. The
flexibility, lack of vendor lock-in, and increased alerting capabilities
of the software RAID resulted in it being chosen for this network.

There are 2 raids configured within the network to manage the storage
of data. The first raid is accessible at /mnt/critical within the
server environment. This serves as the primary storage location for
the services. The second raid is /mnt/backup which is larger than
/mnt/critical and services to store multiple years worth of backups. As mentioned above, these RAIDs are both running in a RAID-1 configuration
with 2 physical disks present in each. This provides redundancy for
disk failure in either of these two raids.

## Setup

These setup notes will walk through the process of setting up a software
RAID using the mdadm package from linux. The first step in setting
up a RAID is to install the necessary packages. This can be
completed with the following commands.

```bash
sudo apt-get update
sudo apt-get install mdadm
```

The next step is to identify which devices within the system need to
be included in the RAID. The *lsblk* command can be utilized to list
all the block devices (hard disks) within the server environment.

```bash
lsblk
```

This command will provide a list of all the devices and their respective sizes.
This information should help to identify the block device names
that will need to be utilized in the next step.

After identifying the devices, the next step is to create the RAID
using those physical devices. The following is an example of that
command for devices /dev/sdb1 and /dev/sdc1.

```bash
mdadm --create --verbose /dev/md/<name> --level=1 --raid-devices=2 /dev/<dev 1> /dev/<dev 2>

Ex.
mdadm --create --verbose /dev/md/critical --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

The above example created what is called a multi-device (md) in the MDADM
terminology. The MD device acts as a virtual device that can be written
to or read from within Linux environment. All actions on the MD device
will span across the physical devices added to the RAID. The name given
to the MD in the above example is */dev/md/critical* as it corresponded
to the critical RAID.

## Checking Status

The status of the RAIDs can be checked through a couple different avenues. The first path is to cat the /proc/mdstat file within the
Linux environment.

The following is an example of the output that becomes available as
part of reading that virtual file.

```bash
cameron@vcenter:~$ cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md126 : active raid1 sdd1[1] sdc1[0]
      3906876416 blocks super 1.2 [2/2] [UU]
      bitmap: 0/30 pages [0KB], 65536KB chunk

md127 : active raid1 sdf1[1] sde1[0]
      976620544 blocks super 1.2 [2/2] [UU]
      bitmap: 0/8 pages [0KB], 65536KB chunk
```

There's a detail command that can be leveraged with *mdadm* to get
even more information on a particular RAID. The following is an example
of looking a bit more in depth at a particular RAID.

```bash
root@vcenter:/home/cameron# mdadm --detail /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Mon Apr 12 11:20:41 2021
        Raid Level : raid1
        Array Size : 976620544 (931.38 GiB 1000.06 GB)
     Used Dev Size : 976620544 (931.38 GiB 1000.06 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Tue Mar 21 21:29:09 2023
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : www:critical
              UUID : 2b068395:2f23a3ef:e6bbe535:51acfb80
            Events : 59106

    Number   Major   Minor   RaidDevice State
       0       8       65        0      active sync   /dev/sde1
       1       8       81        1      active sync   /dev/sdf1
```

As can be seen, there are a lot more details available from the use
of that command. This page will also provide details if the RAID
happens to be going through a re-synchronization process.

It's important to note that the device created with the above commands
does not yet have a file-system or structure to data on it. It's
effectively a RAW block device. As such, it will be necessary to
create a file-system on the md for it to be utilized for storage.

The following in an example of creating a file-system on the MD and
subsequently mounting it to /mnt/critical. However, the following
will not add any sort of encryption on the data stored on the
disks. For encryption, the steps in the LUKS section should be
followed.

```bash
mkfs.ext4 /dev/md/critical
mkdir -p /mnt/critical
mount -t ext4 /dev/md/critical /mnt/critical
```

There are also changes that could be made to /etc/fstab that would
handle the auto-mount of this on the next reboot.

## Maintenance and Performance

There are some configurable options with RAID to keep in mind.

### Check Operations Frequency

The check operations are typically quite resource intensive to RAIDs. The process
involves walking the contents of the RAID and ensuring that all drives maintain the
appropriate data. This can significantly impact the availability of resources and 
response times for the server.

The following command will list the scheduling for when an automated check is performed. Scheduling
for a time with lower usage is ideal from an impact perspective.

```bash
sudo systemctl list-timers mdcheck_start
```

This will result in a message like the following:

```bash
NEXT                        LEFT                LAST                        PASSED            UNIT                ACTIVATES            
Sun 2023-05-07 13:51:06 CDT 4 weeks 0 days left Sun 2023-04-02 07:54:24 CDT 1 week 0 days ago mdcheck_start.timer mdcheck_start.service

1 timers listed.
Pass --all to see loaded but inactive timers, too.
```

The above lists the frequency in which automated check operations are being performed.

### Speeds for Check and Rebuild Operations

There are speed limitations that the software utilizes as a goal when handling RAID
operations. There is a minimum and maximum speed associated with the RAID.

```bash
sudo sysctl dev.raid.speed_limit_min
sudo sysctl dev.raid.speed_limit_max
```

The following is an example of the default values that are configured within the software.

```bash
root@vcenter: sysctl dev.raid.speed_limit_min
dev.raid.speed_limit_min = 1000
root@vcenter: sysctl dev.raid.speed_limit_max
dev.raid.speed_limit_max = 200000
```

The values can be changed using the systemctl command as well. The numbers are in kilobytes/second
on a per-device basis. The following will set a limit of around 7 MB/sec.

```bash
sudo sysctl -w dev.raid.speed_limit_max=7000
```

The following are some values based upon the number of drives. The /etc/sysctl.conf file contains
the default values for the system. The following is an exmaple of the content.

```
#################NOTE ################
##  You are limited by CPU and memory too #
###########################################
dev.raid.speed_limit_min = 50000
## good for 4-5 disks based array ##
dev.raid.speed_limit_max = 2000000
## good for large 6-12 disks based array ###
dev.raid.speed_limit_max = 5000000
```

# Linux Unified Key Setup (LUKS)

Another important decision is to whether the data being written to the
RAID will be protected through the use of encryption. The use of encryption
with the RAID ensures that all data placed on the RAID will not be
readable by unauthorized individuals that may have physical access
to the drives.

Disk encryption within Linux commonly utilizes a combination of
dm-crypt and Linux Unified Key Setup (LUKS). The dm-crypt module
handles the cryptographic operations while LUKS provides the
key management needed for dm-crypt to operate.

## Setup
Integration with RAID devices configured in previous section would enable the ability
to have redundancy and protection of data. Once enabled, a password will need to
be provided each time the RAID is mounted. Enabling protection also means that
it won't be possible for the system to automatically recover in event of power loss.

The following command will initiate the creation process on the /dev/md/critical block
device.

```bash
cryptsetup -y -v luksFormat /dev/md/critical
```

A prompt for password entry will be appear. Be sure to keep the password in safe place
as there is no recovery path if forgotten. Once completed, the RAID device will be
encrypted but there won't be a file-system present. Also, the encrypted volume will
need to be mapped into the file hierarchy before it can be utilized.

The following command will mount the encrypted partition to the provided mapper point. In
this case, the mapper's name is called secret.

```bash
cryptsetup luksOpen /dev/md/critical <mapper_point>

Ex.
cryptsetup luksOpen /dev/md/critical secret
```

By default, the partition creation process does not fill all the drive's contents. This
means that any data previously stored on the drive is still present. This can be alleviated
by fill the secret partition with random data that covers entire drive.

```bash
pv -tpreb /dev/zero | dd of=/dev/mapper/secret bs=128M
```

The filling process will take a long time and will vary depending on the speed and size of the
drives. Finally, the last step in the setup process is to create a file-system on the RAID
device. This will allow for mounting the encryption volume/RAID into the file-system and provide
transparent encryption and redundancy in store.

The following command will create an ext4 file-system within the encrypted partition.

```bash
mkfs.ext4 /dev/mapper/secret
```

Once the file-system is created it will need to be mounted into the file hierarchy.

```bash
mount /dev/mapper/secret /mnt/critical
```

## Unlocking Process
There are 2 steps associated with unlocking an encrypted partition using LUKS.

1. Encrypted Partition needs to be mounted to an accessible block device.
2. File-system on the device must be mounted to become accessible.

The commands seen above show an example of the process but a more targeted combination
are provided for preference.

```bash
# Mount Block Device
cryptsetup luksOpen /dev/md/critical secret

# Mount file-system
mount -t ext4 /dev/mapper/secret /mnt/critical
```

The previous shows the process for making the encrypted partition available for file storage
at the /mnt/critical location.

## Locking Process

The following is the process for locking the RAID devices to prevent data corruption on
power down. The following sequence should be followed when shutting down the system to
ensure that everything remains in consistent state.

```bash
# Unmount the file-system to prevent use
umount /mnt/critical

# Close encrypted partitiont to remove key from memory
cryptsetup luksClose secret
```

## Key Backup

The LUKS package utilizes a series of headers to store the decryption keys in a wrapped
format. The password that is entered is utilized to derive a key that is utilized to
protect the actual encryption key for the partition. This design allows for changing of
password without requiring the entire drive to be re-encrypted. The LUKS header can actually
maintain several different passwords that will unlock the disk. This is done through a series
of slots that can be utilized for storing the wrapped keys.

The following command can be utilized to dump the header associated with the LUKS partition.

```bash
cryptsetup luksDump /dev/mapper/secret
```
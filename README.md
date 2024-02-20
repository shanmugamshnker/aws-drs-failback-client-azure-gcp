

This package includes a shell script to convert the AWS Elastic Disaster Recovery (AWS DRS) provided failback client to a bootable disk that can be attached to a Compute Engine (VM) on Google Cloud Platform (GCP) or Azure. This VM can later be used to initiate the failback process from GCP/Azure to AWS as part of a Disaster Recovery exercise. 


## Description

When testing or performing a disaster recovery solution, it’s often required not only to test the failover process (from primary to DR), but also to spin up the DR site, perform some writes there, and then failback from DR to primary to simulate a full cycle of an outage. This process is called a failback and during which the data replication direction has to be reversed. AWS DRS provides a Failback Client that help with this process. However, when your primary site is cloud, GCP or Azure for example, you can’t directly use that client because you can’t control the booting sequence on a VM on cloud. The only way to do so is by converting the failback Client to a bootable disk that is compatible with the cloud provider that hosts your primary workload. 

This package includes:
* The script that does that convert AWS DRS Failback Client livecd to a bootable disk on GCP <Convertor-GCP.sh>


## Conversion workflow

![conversion](https://user-images.githubusercontent.com/59539231/214702878-2614d44b-0bf0-4b21-9bb7-4a200fd7306f.png)



## Step by Step 
1. Set Up VMware Workstation: Install VMware Workstation to create a virtual environment. Ensure this virtual machine (VM) has internet access to download the required ISO and Kernel.
3. Download Amazon Linux 2 ISO: Access the Amazon Linux 2 VM User Guide and obtain the Amazon Linux 2 ISO file. Use the link below
  - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-2-virtual-machine.html
3. Create New VM: Use the downloaded ISO to create  a new VM in VMware Workstation.
4. Prepare Additional Storage: In the Amazon Linux 2 VM, Provision a 30 GiB block storage volume and attach it to the VM. This storage is utilized for storting the ISO image and installing kernel and grub.
```
# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdb    202:16   0  30G  0 disk
```
5. Partition the /dev/xvdb device and format it using a file system of your choice.
```
# fdisk -c /dev/xvdb

Welcome to fdisk (util-linux 2.30.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xb3d3b77d.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-62914559, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-62914559, default 62914559):

Created a new partition 1 of type 'Linux' and of size 30 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
```
# mkfs -t xfs /dev/xvdb1
meta-data=/dev/xvdb1             isize=512    agcount=4, agsize=1966016 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=7864064, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=3839, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
```
# blkid
/dev/xvda1: LABEL="/" UUID="43213978-8f11-4c09-8bae-ac42538542f2" TYPE="xfs" PARTLABEL="Linux" PARTUUID="ef89ea91-0190-487b-a64e-91e3c1e4341a"
/dev/xvdb1: UUID="33a2073d-a4a6-46c1-86b3-18e947ad651d" TYPE="xfs" PARTUUID="b3d3b77d-01"
```

6. Download the Convertor Script: Retrieve the script from the provided URL:
`wget https://raw.githubusercontent.com/shanmugamshnker/aws-drs-failback-client-azure-gcp/document_fix/Convertor-GCP.sh`

7.Grant executable permissions to the downloaded script using the command:
`# chmod a+x Convertor-GCP.sh`

8. Execute the Script: Run the script by executing:
`# ./Convertor-GCP.sh`

The script will prompt you to enter several details:

Region: Choose any available region (e.g., "us-east-1"). 
Disk Name: Specify the disk name (e.g., "/dev/sda"). 
Partition Name: Indicate the partition name (e.g., "/dev/sda1").

```
# ./Convertor-GCP.sh
Enter the region name:
us-east-1
Enter the Disk name i.e /dev/sda :
/dev/xvdb
Enter the Partition name i.e /dev/sda1 :
/dev/xvdb1
```

- Script started downloading the image and start installing kernel and grub in /dev/xvdb, The output is huge hence it truncketed. 

```
Downloading Failback Client ISO
--2024-02-20 17:09:01--  https://aws-elastic-disaster-recovery-us-east-1.s3.amazonaws.com/latest/failback_livecd/aws-failback-livecd-64bit.iso
Resolving aws-elastic-disaster-recovery-us-east-1.s3.amazonaws.com (aws-elastic-disaster-recovery-us-east-1.s3.amazonaws.com)... 16.182.39.153, 52.216.177.35, 52.216.221.145, ...
Connecting to aws-elastic-disaster-recovery-us-east-1.s3.amazonaws.com (aws-elastic-disaster-recovery-us-east-1.s3.amazonaws.com)|16.182.39.153|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 953081856 (909M) [binary/octet-stream]
Saving to: ‘aws-failback-livecd-64bit.iso’

100%[=============================================================================================>] 953,081,856 30.6MB/s   in 32s

2024-02-20 17:09:33 (28.5 MB/s) - ‘aws-failback-livecd-64bit.iso’ saved [953081856/953081856]
[....]
ounting /proc, /sys and /dev in /secondery_root
mount: /proc bound on /secondery_root/proc.
mount: /sys bound on /secondery_root/sys.
mount: /dev bound on /secondery_root/dev.
installing kernel
warning: Unable to get systemd shutdown inhibition lock
warning: Unable to get systemd shutdown inhibition lock
Preparing...                          ################################# [100%]
Updating / installing...
   1:kernel-4.14.268-205.500.amzn2    ################################# [100%]
bash
modsign
nss-softokn
i18n
drm
plymouth
bcache
crypt
dm
dmraid
dmsquash-live
kernel-modules
lvm
mdraid
resume
rootfs-block
terminfo
udev-rules
systemd
usrmount
base
fs-lib
img-lib
microcode_ctl-fw_dir_override
shutdown
grubby fatal error: unable to find a suitable template
grubby fatal error: unable to find a suitable template
generating grub2.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.10.209-198.812.amzn2.x86_64
Found initrd image: /boot/initramfs-5.10.209-198.812.amzn2.x86_64.img
Found linux image: /boot/vmlinuz-4.14.336-253.554.amzn2.x86_64
Found initrd image: /boot/initramfs-4.14.336-253.554.amzn2.x86_64.img
Found linux image: /boot/vmlinuz-4.14.268-205.500.amzn2.x86_64
Found initrd image: /boot/initramfs-4.14.268-205.500.amzn2.x86_64.img
done
installing grub
Installing for i386-pc platform.
Installation finished. No error reported.
```

9. Set Root Login Password for ISO Image: Once the conversion script has executed successfully without any errors, establish the root login password for the ISO image with these commands
```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      8.0G  2.8G  5.3G  34% /
/dev/loop0      909M  909M     0 100% /mnt
/dev/loop1      791M  791M     0 100% /squashfs
/dev/loop2      3.8G  2.0G  1.6G  56% /rootfs
/dev/xvdb1       30G  2.4G   28G   8% /secondery_root
```

Follow the prompts to set a secure root password of your choice.

```
# chroot /secondery_root/
# passwd
```


## For Azure

* Convert the results file from step 8 into VHD using the following steps

* Using Microsoft Virtual Machine Converter 3.0 on Windows machine >> Launch PowerShell and load the module 
MvmcCmdlet.psd1 ‘path to the module’


* Convert VMDK to VHD using the command 
ConvertTo-MvmcVirtualHardDisk – sourceLiteralpath “path to your file”


* You may encounter an error that says “The entry 0 is not a supported disk database entry for the descriptor”. 

<img width="858" alt="error" src="https://user-images.githubusercontent.com/59539231/214702970-a8c62e65-17dc-4adf-a245-374646401340.png">


The issue is caused by VMDK file descriptor entries that Microsoft Converter doesn't recognize. To resolve the issue, we need to remove these non-recognized entries. Download and extract dsfok (https://www.mysysadmintips.com/-downloads-/Windows/Servers/dsfok.zip)tool. This tool helps to extract descriptor from VMDK file, and then save modified version back into the VMDK.

<img width="858" alt="dsfo" src="https://user-images.githubusercontent.com/59539231/214703089-130fc2f8-8a58-4dca-bc92-cdc3a69c1ae2.png">


You can find the “descriptor.txt” in the location you have specified in the above command. The highlighted entry is not a supported disk database. You need to comment it. 

<img width="1064" alt="fix" src="https://user-images.githubusercontent.com/59539231/214703134-866110ad-0a2d-4d59-b2d5-6e1aefb79de5.png">


Add the descriptor back to VMDK

<img width="862" alt="dsfi" src="https://user-images.githubusercontent.com/59539231/214703169-30c1bcdf-f7f6-42d4-8721-8e5c7724825d.png">


* The last step is to upload the VHD file to Azure > create a snapshot from it > use the snapshot to boot the final VM. 

## For GCP 
   
* Once completed, find the resulted .vmdk file in the VMWare workstation and upload it to GCP Cloud Storage. 
* Follow the instuctions in the blog post to create and image and continue the failback replication. The blog post link is here 


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.


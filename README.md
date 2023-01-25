

This package includes a shell script to convert the AWS Elastic Disaster Recovery (AWS DRS) provided failback client to a bootable disk that can be attached to a Compute Engine (VM) on Google Cloud Platform (GCP) or Azure. This VM can later be used to initiate the failback process from GCP/Azure to AWS as part of a Disaster Recovery exercise. 


## Description

When testing or performing a disaster recovery solution, it’s often required not only to test the failover process (from primary to DR), but also to spin up the DR site, perform some writes there, and then failback from DR to primary to simulate a full cycle of an outage. This process is called a failback and during which the data replication direction has to be reversed. AWS DRS provides a Failback Client that help with this process. However, when your primary site is cloud, GCP or Azure for example, you can’t directly use that client because you can’t control the booting sequence on a VM on cloud. The only way to do so is by converting the failback Client to a bootable disk that is compatible with the cloud provider that hosts your primary workload. 

This package includes:
* The script that does that convert AWS DRS Failback Client livecd to a bootable disk on GCP <Convertor-GCP.sh>


## Conversion workflow

![conversion](https://user-images.githubusercontent.com/59539231/214702878-2614d44b-0bf0-4b21-9bb7-4a200fd7306f.png)



## Step by Step 

1. Download the AWS Failback Client ISO 
   https://aws-elastic-disaster-recovery-{REGION}.s3.amazonaws.com/latest/failback_livecd/aws-failback-livecd-64bit.isoand 
2. Boot the Failbacl Client in VMware Workstaion.  
4. The Failbacl Client will prompt for the AWS Region to initiate failback. Press "Ctrl+c" and you will be dropped into the ec2-user home directory. To switch to root use "sudo -i".
5. Download the script <Convertor-GCP.sh> 
6. Add executable permission to the script using "chmod a+x <Convertor-GCP.sh>
7. Run the script script ./<Convertor-GCP.sh>. You need to provide: 
      - Region (you can use any region i.e. "us-east-1")
      - Disk name (i.e /dev/sda)
      - Partation name  (i.e /dev/sda1)
   
   The script will install the kernel, generate grub.cfg, and install grub on the disk. The steps from are different between GCP and Azure. 
   
## For Azure

* Convert the results file from step 7 into VHD using the following steps

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


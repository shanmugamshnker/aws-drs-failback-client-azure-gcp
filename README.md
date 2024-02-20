

This package includes a shell script to convert the AWS Elastic Disaster Recovery (AWS DRS) provided failback client to a bootable disk that can be attached to a Compute Engine (VM) on Google Cloud Platform (GCP) or Azure. This VM can later be used to initiate the failback process from GCP/Azure to AWS as part of a Disaster Recovery exercise. 


## Description

When testing or performing a disaster recovery solution, it’s often required not only to test the failover process (from primary to DR), but also to spin up the DR site, perform some writes there, and then failback from DR to primary to simulate a full cycle of an outage. This process is called a failback and during which the data replication direction has to be reversed. AWS DRS provides a Failback Client that help with this process. However, when your primary site is cloud, GCP or Azure for example, you can’t directly use that client because you can’t control the booting sequence on a VM on cloud. The only way to do so is by converting the failback Client to a bootable disk that is compatible with the cloud provider that hosts your primary workload. 

This package includes:
* The script that does that convert AWS DRS Failback Client livecd to a bootable disk on GCP <Convertor-GCP.sh>


## Conversion workflow

![conversion](https://user-images.githubusercontent.com/59539231/214702878-2614d44b-0bf0-4b21-9bb7-4a200fd7306f.png)



## Step by Step 

# Setting up a VMware Environment with Amazon Linux 2 To create Failback ISO for Azure and GCP

1. Install VMware Workstation: Begin by setting up a VMware environment, such as VMware Workstation. This will serve as the platform for creating and managing your virtual machines.
2. Download Amazon Linux 2 ISO: Visit the Amazon Linux 2 VM User Guide and download the Amazon Linux 2 ISO file.
3. Use the downloaded ISO to create a new virtual machine in your VMware Workstation.
4. After setting up the Amazon Linux 2 VM, create an additional block storage volume of 30 GiB and attach it to the VM created in Step 2. We use this volume to install the kernel and grub.
5. Download the script from the following URL:
`wget https://raw.githubusercontent.com/shanmugamshnker/aws-drs-failback-client-azure-gcp/document_fix/Convertor-GCP.sh`
6. Grant executable permissions to the downloaded script using the command:
`# chmod a+x Convertor-GCP.sh`
7. Execute the Script: Run the script by executing:
`# ./Convertor-GCP.sh`

The script will prompt you to enter several details:

Region: Choose any available region (e.g., "us-east-1").
Disk Name: Specify the disk name (e.g., "/dev/sda").
Partition Name: Indicate the partition name (e.g., "/dev/sda1").



The script will install the kernel, generate grub.cfg, and install grub on the disk. The steps from are different between GCP and Azure. 
   
## For Azure

* Convert the results file from step 6 into VHD using the following steps

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


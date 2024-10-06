# Demo: Configuring EBS volumes in AWS

## Overview
In this Demo, I interact with Elastic Block Store (EBS) and EC2 instances in AWS. The goal is to move volumes between instances. Here's what I will accomplish:

- Create and mount an EBS volume to an EC2 instance.
- Migrate the volume to another instance and verify the data.

By the end of this Demo, I will have hands-on experience managing storage in AWS EC2.


## One-Click Deployment
Before starting, I use the following one-click deployment link to automatically set up the necessary AWS resources. This link provisions 3 public EC2 instances and an 8 GB boot volume for each in preparation for the demo:

[Click here for One-Click Deployment](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0004-aws-associate-ec2-ebs-demo/A4L_VPC_3PUBLICINSTANCES_AL2023.yaml&stackName=EBSDEMO)

- ![stack](https://github.com/user-attachments/assets/25e86463-75d8-4ee3-807b-e9c1675a1fbc)

## Prerequisites

- AWS Account with appropriate IAM permissions to work with EC2, EBS, and snapshots (in my case the management account used in my other demos).

## Steps

### Step 1: Create and Attach an EBS Volume
- Created a new 10 GB gp3 EBS volume in the availability zone us-east-1a (AZ A) and added a tag to identify the volume (Key: Name, Value: EBSTestVolumeFarah) & clicked Create Volume.

- ![volume-created](https://github.com/user-attachments/assets/e14b0447-ff52-47b6-b0a3-43ff8b657a8d)

- When the volume is in the available state, attached it to the A4L-EBS-INSTANCE1-AZA instance and selected the device name /dev/sdf.

- ![available](https://github.com/user-attachments/assets/5c733730-057b-41ce-91f3-413600ff0f56)

- ![attach-volume](https://github.com/user-attachments/assets/42be9b16-63e2-4f1f-b4ed-8eca972b27e8)

- Now the volume EBSTestVolumeFarah is in-use state

- ![Capture d'écran 2024-10-06 144614](https://github.com/user-attachments/assets/4e46a3de-256b-4394-8d91-6c3089bd5336)

### Step 2: Connect to EC2 Instance and Prepare the EBS Volume
- Connected to the A4L-EBS-INSTANCE1-AZA instance using EC2 Instance Connect.

- ![instance-connect](https://github.com/user-attachments/assets/2d659722-80ae-416d-a7dc-c53ebb3dcbe5)


- Listed block devices:
lsblk
- Checked for existing file systems on the block device:
sudo file -s /dev/xvdf
Note: There is no file system present.
- Created a new file system:
sudo mkfs -t xfs /dev/xvdf
sudo file -s /dev/xvdf

- ![fs](https://github.com/user-attachments/assets/f7dd2894-3236-4af4-8008-2675da84b19c)

- Created a mount point:
sudo mkdir /ebstest
- Mounted the EBS volume:
sudo mount /dev/xvdf /ebstest
- Navigated to the mount point:
cd /ebstest
- Created a test file:
sudo nano testfile.txt
Added the message "the file system works"
- Verified the file creation:
ls -la
- Rebooted the A4L-EBS-INSTANCE1-AZA instance:
sudo reboot

- ![testfile reboot](https://github.com/user-attachments/assets/6010ac93-0411-49cd-8a32-c96f4eeb7002)


- reconnected using EC2 Instance Connect and checked file systems:
df -k
Note: The file system isn't displayed because I have manually mounted the file system before the reboot and need to make it automatically mounted at boot.
- Listed block device UUIDs:
sudo blkid
-Edited the fstab file to ensure the filesystem is automatically mounted at boot (because I mounted it manually at first which is not convenient) and I added the following line:
UUID=c46fa933-0eeb-47f9-b3c9-c260bf76e5d3  /ebstest  xfs  defaults,nofail
sudo nano /etc/fstab
- Remounted all filesystems:
sudo mount -a
- Verified the file still exists and re-checked disk space where we can see the /dev/xvdf file system:
cd /ebstest
ls -la
df -k

- ![df-k](https://github.com/user-attachments/assets/1869d477-bad4-4aa5-8583-abf37a9e158e)



### Step 3: Migrate EBS Volume to Another Instance

- Stopped the A4L-EBS-INSTANCE1-AZA instance.

- ![stopped](https://github.com/user-attachments/assets/c2d40208-13e7-4a35-9acc-b9c770273e66)

- Detached the EBS volume from A4L-EBS-INSTANCE1-AZA instance.
- Attached the EBS volume to A4L-EBS-INSTANCE2-AZA instance.
- Connected to Instance 2 and ran the following commands to verify the volume is mounted correctly which is the case because we can see the testfile.txt:

lsblk
sudo file -s /dev/xvdf
sudo mkdir /ebstest
sudo mount /dev/xvdf /ebstest
cd /ebstest
ls -la

- ![ebs](https://github.com/user-attachments/assets/9319fb35-a6e7-4848-b3df-6e38199b1f6d)

## Conclusion

This demo provided hands-on experience with managing EBS volumes in AWS. By following the steps, I successfully created, attached, and migrated an EBS volume between EC2 instances. The persistence of data in EBS volumes was demonstrated, as well as how volumes can be moved seamlessly between instances in the same availability zone. Additionally, the importance of configuring automatic mounting at boot using the `fstab` file was covered.

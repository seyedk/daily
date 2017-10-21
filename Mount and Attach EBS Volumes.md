In this post, I'm going to spin up a new EBS volume that I intend to use it as my local Archive drive.

first, from within the aws management console, I create a new volume in the same region-AZ that my ec2 instance is at:

```
aws ec2 create-volume --size 20 --region us-west-2 --availability-zone us-west-2b --volume-type gp2 --tag-specifications 'ResourceType=volume,Tags=[{Key=purpose,Value=Archive},{Key=Usage,Value=ADS}]'

```
if you see a response like this, then you're good!
```
{
    "AvailabilityZone": "us-west-2b",
    "CreateTime": "2017-09-09T16:08:50.570Z",
    "Encrypted": false,
    "Size": 20,
    "SnapshotId": "",
    "State": "creating",
    "VolumeId": "vol-05XXXXXXXXXX",
    "Iops": 100,
    "Tags": [
        {
            "Key": "purpose",
            "Value": "Archive"
        },
        {
            "Key": "Usage",
            "Value": "ADS"
        }
    ],
    "VolumeType": "gp2"
}
```
now you should be able to see the volue in the EC2 console/Volumes.

next step is to attach the volume to your instance from within either the console or from the EC2 CLI. I rather use CLI. make a note of the volume ID and the instance ID you want to attach:

```
aws ec2 attach-volume --volume-id <volume-id> --instance-id <instance-id> --device /dev/xvdh
```

you should get a response like the one below:
```
{
    "AttachTime": "2017-09-09T17:40:02.162Z",
    "Device": "/dev/xvdh",
    "InstanceId": "<instance-id>",
    "State": "attaching",
    "VolumeId": "<volume-id>"
}
```
While the volume is getting attached, try to ssh to your linux instance.

first make sure you can see the volume available.Use the lsblk command for this and its mount point (this command removes the `/dev/` prefix)

```
 lsblk
 -----------
NAME  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvdf  202:80   0  100G  0 disk
xvda1 202:1    0    8G  0 disk /
xvdh    202:112  0  20G  0 disk
```

you need to create a file system on the volume. *make sure you're doing this on a raw block device. to check that use the following comamnd:*

```
file -s /dev/xvdh
```

The output of the command should be `/dev/xvdh: data`. In case you see something similar to `/dev/xvda1: Linux rev 1.0 ext4 filesystem data, UUID=1701d228-e1bd-4094-a14c-8c64d6819362 (needs journal recovery) (extents) (large files) (huge files)` **then something's wrong**


if safe, then create a file system of type ext4 (you may choose a different fs).

``` 
mkfs -t ext4 /dev/xvdh
```

Next, mount the device to a mount point. 
1. create a mount point
```
mkdir /ads-archive/
```
2. mount the device to the mount point
```
mount /dev/xvdh /ads-archive/
```

## Mount on every REBOOT
to mount the new EBS volume on every system reboot, add an entry for the device to the /etc/fstab file
1. backup your /etc/fstab
```
cp /etc/fstab/ /etc/fstab.bak
```
2. add the following (or similar entry to the fstab)
```
/dev/xvdh   /ads-archive        ext4    defaults,nofail 0   2
```
3. run mount to mount all devices
```
mount -a
```

Now use the drive: just move your log files to the archive directory!

References
1) [Make volumes available for use][1]
2) [ec2 create volume cli][4]
3) [ec2 attach volume cli][3]
4) [ec2 cli][2]

[1]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html "usevol :vol"

[2]: http://docs.aws.amazon.com/cli/latest/reference/ec2/index.html#cli-aws-ec2 "ec2cli: ec2cli"

[3]: http://docs.aws.amazon.com/cli/latest/reference/ec2/attach-volume.html "ec2cli: attachVolume"

[4]: http://docs.aws.amazon.com/cli/latest/reference/ec2/create-volume.html "ec2cli: createVolume"


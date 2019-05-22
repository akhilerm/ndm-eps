### 1. Implementation of BlockDevice(BD) resource

A BlockDevice (BD) is defined as any physical/logical device that is available on the host. Physical disks, virtual disks, partitions, lvms, RAID devices all can be categorized as block devices.

```
sda           8:0    0 465.8G  0 disk 
├─sda1        8:1    0   512M  0 part /boot/efi
└─sda2        8:2    0 465.3G  0 part /
sdb           8:31   1  14.4G  0 disk 
sdc           8:32   1  14.4G  0 disk 
├─sdc1        8:33   1     4G  0 part 
│ └─vg0-lv0 253:0    0     6G  0 lvm  
└─sdc2        8:34   1     4G  0 part 
  └─vg0-lv0 253:0    0     6G  0 lvm  
```
A node having the above disk configuration will have a total of 8 blockdevices:
	1. /dev/sda
	2. /dev/sdb
	3. /dev/sdc
	4. /dev/sda1
	5. /dev/sda2
	6. /dev/sdc1
	7. /dev/sdc2
	8. /dev/mapper/vg0-lv0 --> /dev/dm-0

### 2. What happens to Disk resource

Disk resource will be created only for physical disks which are attached to the host. Virtual disks, logical volumes etc will have only a BD resource. Disk resource will have information of the BlockDevice which is backed by the respective disks. Physical attributes of the disk like temperature will remain inside the Disk resource and all other details will be eventually phased out.


### 3. Implementation of BlockDeviceClaim(BDC) resource

BDC-BD will be a one - one mapping like PVC-PV. Whenever a BD is required by the upper layer (Maya), a BDC is created. The BDC resource is being watched by the NDM-Operator. NDM operator checks for available BDs that satisfy the requirements in the BDC and mark the BDC as Bound and BD as Claimed. On deleting the BDC, the BD will be marked Unclaimed and can be used by other BDCs.
### Support for LVM in Node Device Manager

For every LVM device created in the host, we need to find, the volume group, physical volumes backing the volume group and update that info into the disk CR.

##### Workflow for LVM devices
1. A udev event will be generated for `/dev/dm-X`
2. From the udev probe, the following information will be obtained. DM_VG_NAME(vg0) and DM_NAME(vg0-lv0) will give the volume group name and device name. `/dev/mapper/DM_NAME` will be the device path. 

(Due to this [issue](https://github.com/moby/moby/issues/16160) in docker, this path will not be available inside the container)

3. We need to find the backing devices for this Logical Volume and mark this as an aggregate device (TODO: need to be clear on how aggregates are managed)

4. For finding the backing devices, an lvm package can be introduced which gives the list of backing devices given a volume group/dm device. The code can make use of `/etc/lvm/backup/DM_VG_NAME` file and read its contents to get the PVs and LVs in the volume group.


**NOTE:** The reason for selecting `/dev/mapper/DM_NAME` as path instead of `/dev/dm-X` is because, when the device is mounted, the path used in `/proc/1/mounts` is the mapper path. This will help to easily match the device and mountpoint.

I also propose, a `by-uuid` type in devlinks which can be useful in some cases like LVM.

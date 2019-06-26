### UUID generation for disks


Disk UUID generation to uniquely identify a given disk(any-type) in a cluster.


#### Approach

1. If `by-id` is present in Devlink, we go with the current approach. 
   
   If `by-id` is not present, 
Create a GPT partition on that disk, that spans the entire disk. We can use the UUID generated during this time for the blockdevice, and give the parititon to the user.
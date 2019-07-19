### Refactoring the Daemonset code

The current NDM daemonset has many code blocks which are repeated. This can be made much leaner. 
The changes I propose are:

1. Getting rid of DeviceInfo struct. I think, we can use the original DiskInfo struct itself for creating both Disk and BlockDevice CR. Another
struct for keeping DeviceInfo is not required as it is anyway generated from the DiskInfo.

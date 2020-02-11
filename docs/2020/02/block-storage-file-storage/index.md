# Block Storage and File Storage


## Block Storage
1. tgt framework could be used to implement a tgt server in user space.
2. You could customize a new backend store type which could, for example, send the SCSI message in customized format via a socket. Then a user space program listening on the socket could deserialize from customized format and do some additional tasks on the SCSI message like multi-write them to different replicas.
    
    The customized tgt backend store: https://github.com/rancher/tgt and https://github.com/rancher/liblonghorn.



## File Storage

1. Use FUSE

FUSE (Filesystem in Userspace) is a Linux kernel filesystem that  sends the incoming requests over a file descriptor to userspace. FUSE is a protocol.

![](/forgetful/images/block-storage-file-storage-fuse-diagram.png)

There is a FUSE framework in golang: https://github.com/bazil/fuse.

And have a try on a read-only filesystem exported from a zip file:  https://github.com/Murray-LIANG/zipfs.


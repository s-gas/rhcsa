# Block Devices

Block devices are files that provide access to storage devices. The `/dev` directory stores these block device files, which are created automatically by the kernel when it detects a device.

```bash
[root@rocky ~]$ ls -l /dev/sda
brw-rw----. 1 root disk 8, 0 Oct  9 18:02 /dev/sda
```

Block devices are identified with `b`.

Block devices are exposed by the kernel as an array of fixed-size blocks (sectors), which are usually 512 bytes or 4KiB, that can be read or written.

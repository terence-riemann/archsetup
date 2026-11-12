To read USBs on Arch, there are many applications that help do this. When your download is barebones, there is a way to do it manually:

First, find the usb (and its partition(s)) using lsblk:
```bash
lsblk -f
```
If it's not made already, make a usb directory in your mount directory (as we will mount the partition into that directory). You can do this step anyway if you're unsure:
```bash
sudo mkdir -p /mnt/usb
```
Now mount the usb (using the partition identifier name) onto the directory (in my case, my usb (really the usb's partition name) is named sdb1):
```bash
sudo mount /dev/sdX1 /mnt/usb
```
You can access the contents of the usb in the directory /mnt/usb now. You can check its contents by:
```bash
ls /mnt/usb
```
When you are done, you need to unmount the drive before removing the usb:
```bash
sudo umount /mnt/usb
```

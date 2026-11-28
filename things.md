# Manually Reading USBs
To read USBs on Arch, there are many applications that help do this. When your archlinux install is barebones, there is a way to do it manually:

First, find the usb (and its partition(s)) using lsblk:
```bash
lsblk -f
```
If it's not made already, make a usb directory in your mount directory (as we will mount the partition into that directory). You can do this step anyway if you're unsure:
```bash
sudo mkdir -p /mnt/usb
```
Now mount the usb (using the partition identifier name) onto the directory (in my case, my usb (really the usb's partition name) is named sdb1) and grant necessary permissions to write onto the usb for the user:
```bash
# -o - options, gid = users - the group ownership of files on the mounted filesystem, fmask=113,dmask=002 - setting specific permissions to the group, /dev/sdb1 - usb partition id, /mnt/usb - directory being mounted onto.
mount -o gid=users,fmask=113,dmask=002 /dev/sdb1 /mnt/usb
```
You can access the contents of the usb in the directory /mnt/usb now. You can check its contents by:
```bash
ls /mnt/usb
```
When you are done, you need to unmount the drive before removing the usb:
```bash
sudo umount /mnt/usb
```
# Configuring hibernation
When putting the laptop to sleep, the session is frozen on your ram and the laptop goes into a low power mode. For hibernation, the session stored on your ram is saved onto your disk and the laptop is completely powered off. In order to be able to hibernate the laptop, there must be somewhere on the disk that can accommodate for the hibernation image. This is usually the swap partition, but ours is 4gb. It is too much to manually change the partitions to make the swap partition bigger so instead we will create a 'swap' file on the root partition that will store our hibernation image. Since our ram is 32gb, typically you would want a 32gb swap file on root, but since we will likely never have such high ram usage, I will opt for a 24gb swap file on root. My root partition as is, is 60gb with 15gb currently being used:
```bash
df -h /
```
Even though my /home directory is sufficiently large to host a 32gb swap file, the /home directory would need access to things deep into the system which we can grant but would cause a security risk if compromised. 
```bash
# create it
sudo fallocate -l 24G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
#verify it
swapon --show
# Make the file persistent
# Open
sudo nvim /etc/fstab
# Add in nvim : /swapfile none swap defaults 0 0
# For chatGPT
findmnt -no UUID -T /swapfile
sudo filefrag -v /swapfile
grep '^HOOKS=' /etc/mkinitcpio.conf
grep '^GRUB_CMDLINE_LINUX_DEFAULT=' /etc/default/grub
stat -f -c 'Filesystem block size: %S' /
getconf PAGESIZE
swapon --show
# Configure GRUB
sudo nvim /etc/default/grub
# Edit in nvim: Change GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet" to GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet resume=UUID=ae66e9e8-5f73-4d1e-a2ce-8ac1e09133a8 resume_offset=14551040"
# Regenerate GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg
# 'Regenerate the initramfs for completeness'
sudo mkinitcpio -P
# Reboot and reverify
sudo reboot
cat /proc/cmdline
# You should see both resume= and resume_offset=
cat /sys/power/resume
cat /sys/power/resume_offset
# Should see the physical_offset of extent 0 from sudo filefrag -v /swapfile
# Test hibernation
systemctl hibernate
# If it fails check:
journalctl -b -u systemd-hibernate.service
```

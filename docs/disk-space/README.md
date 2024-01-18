## Verify the disk space on server and adjust (In this example it will be 2 TB disk space setup)

Login to ubuntu server with the user created in above step (ocpadmin)

```sh
ssh ocpadmin@your_server_ip_address
```

login with sudo access

```sh
sudo su -
```

You will run a test first to verify that it can resize properly before you actually modify the partition.

run below command to verify the disk space

```sh
lsblk
```

Example Outout

```sh
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 63.3M  1 loop /snap/core20/1828
loop1                       7:1    0 91.9M  1 loop /snap/lxd/24061
loop2                       7:2    0 49.9M  1 loop /snap/snapd/18357
loop3                       7:3    0 40.9M  1 loop /snap/snapd/20290
loop4                       7:4    0 63.5M  1 loop /snap/core20/2015
sda                         8:0    0  1.8T  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0  1.8T  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0  100G  0 lvm  /
sdb                         8:16   0  1.8T  0 disk 
```

In above output, we can see the Main Operating system have sda3 partition with total 1.8 TB disk space, Hwever 100 GB is allocated to  "sda3 - ubuntu--vg-ubuntu--lv"

Verify how much disk already utilized (You may need to change below command based on your volume name)

```sh
df -h | grep "ubuntu--vg-ubuntu--lv"
```

Example Outout

```sh
/dev/mapper/ubuntu--vg-ubuntu--lv   98G   24G   70G  26% /
```

Now that we know the logical volume is only 100 Gigs, we can resize it to the remaining open free space. First, we will test this command using the -t switch (for test) and view the output for errors. If clean, proceed.

```sh
lvresize -t -v -l +50%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

Example Outout

```sh
TEST MODE: Metadata will NOT be updated and volumes will not be (de)activated.
  Converted 50%FREE into at most 225410 physical extents.
  Test mode: Skipping archiving of volume group.
  Extending logical volume ubuntu-vg/ubuntu-lv to up to <980.51 GiB
  Size of logical volume ubuntu-vg/ubuntu-lv changed from 100.00 GiB (25600 extents) to <980.51 GiB (251010 extents).
  Test mode: Skipping backup of volume group.
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
```

In above output, Once you see that the test was successful, remove the -t switch (for test) from the previous command to actually increase the logical volume.

```sh
lvresize -v -l +50%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

Example Outout

```sh
Converted 50%FREE into at most 225410 physical extents.
  Creating directory "/etc/lvm/archive"
  Archiving volume group "ubuntu-vg" metadata (seqno 2).
  Extending logical volume ubuntu-vg/ubuntu-lv to up to <980.51 GiB
  Size of logical volume ubuntu-vg/ubuntu-lv changed from 100.00 GiB (25600 extents) to <980.51 GiB (251010 extents).
  Loading table for ubuntu--vg-ubuntu--lv (253:0).
  Suspending ubuntu--vg-ubuntu--lv (253:0) with device flush
  Resuming ubuntu--vg-ubuntu--lv (253:0).
  Creating directory "/etc/lvm/backup"
  Creating volume group backup "/etc/lvm/backup/ubuntu-vg" (seqno 3).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
```

In above output, we can see the logicl volume got successfully resized

run below command to verify the disk space

```sh
lsblk
```
Example Outout

```sh
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0  63.3M  1 loop /snap/core20/1828
loop1                       7:1    0  91.9M  1 loop /snap/lxd/24061
loop2                       7:2    0  49.9M  1 loop /snap/snapd/18357
loop3                       7:3    0  40.9M  1 loop /snap/snapd/20290
loop4                       7:4    0  63.5M  1 loop /snap/core20/2015
sda                         8:0    0   1.8T  0 disk 
├─sda1                      8:1    0     1M  0 part 
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0   1.8T  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 980.5G  0 lvm  /
sdb                         8:16   0   1.8T  0 disk 
```

In above output, we can see logical volume got resize to 980 GB

Now that the logical volume is 980G, get the FS type and resize the filesystem on the newly acquired space. We can see that it’s using ext4.

```sh
df -h -T |grep vg
```

Example Outout

```sh
/dev/mapper/ubuntu--vg-ubuntu--lv ext4       98G   24G   70G  26%
```

Since the file system is ext4, we will use the resize2fs command.

```sh
resize2fs -p /dev/mapper/ubuntu--vg-ubuntu--lv
```

Example Outout

```sh
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required
old_desc_blocks = 13, new_desc_blocks = 123
The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 257034240 (4k) blocks long.
```

Now verify the new size of your root volume to confirm that it has increased in size.

```sh
df -h -T |grep vg
```
Example Output

```sh
/dev/mapper/ubuntu--vg-ubuntu--lv ext4      965G   24G  901G   3% /
```




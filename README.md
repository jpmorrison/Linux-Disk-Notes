
## Cloning

- duplicate a drive
- migrate an old drive to a larger/faster drive

  `ddrescue /dev/sdold /dev/sdnew rescue.log `
  

### Randomize UUIDs 

- Low level copying  (i.e. ddrescue) preservces *everything*. A cloned drive has identical disk and partition UUIDs.
- In general do  *not* randomize, espcially if you are just creating a backup drive or replacing an existing drive
- If you want to re-use the original drive, consider erasing/wiping it.
- If you do not want to wipe the drive, you probably need to randomize disk and/or partition UUIDs due to conflicts:
  - Windows ignores a cloned basic GPT disk (AHCI). Dynamic disks will be offline.
  - Linux may have incorrect mounts because of UUID= entries in /etc/fstab

#### Randomize disk UUID

This only works for GPT disks

`gdisk /dev/cloned`

Type x for expert mode then g for 'change disk GUID'. You can enter a new GUID or type R to generate a random one.

```
gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.


Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu

Command (? for help): x

Expert command (? for help): ?
a	set attributes
c	change partition GUID
d	display the sector alignment value
e	relocate backup data structures to the end of the disk
f	randomize disk and partition unique GUIDs
g	change disk GUID
h	recompute CHS values in protective/hybrid MBR
i	show detailed information on a partition
j	move the main partition table
l	set the sector alignment value
m	return to main menu
n	create a new protective MBR
o	print protective MBR data
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	resize partition table
t	transpose two partition table entries
u	replicate partition table on new device
v	verify disk
w	write table to disk and exit
z	zap (destroy) GPT data structures and exit
?	print this menu

Expert command (? for help): g
Enter the disk's unique GUID ('R' to randomize): 

```

type w to write the changed disk GUID

#### new UUID on cloned EFI partition 

Linux systems typically mount the EFI partition on /boot/efi with a UUID= entry in /etc/fstab

If you keep the cloned drive in the same systen, the UUID in /dev/disk/by-uuid/ may point to the wrong EFI partition. Randomizing disk/partition UUIDs doesn't help - you need to give the EFI parition a new FAT serial number:

Use mlabel on the EFI partition you want to update:

mlabel -i /dev/sda1 -n 

Run `partprobe` so the kernel updates the partition info and /dev/disk/

Now you'll two different UUID entries for the EFI partitions

```
ls -ln /dev/disk/by-uuid/ | grep sd[ab]1

lrwxrwxrwx 1 0 0 10 Dec 12 18:32 1BDB-B012 -> ../../sda1
lrwxrwxrwx 1 0 0 10 Dec 12 18:32 E00F-EA98 -> ../../sdb1

```

- Edit the UUID= entry in /etc/fstab if needed
- May need to run `systemctl daemon-reload` due to systemd weirdness (boot.mount)
- mount /boot/efi
- update-grub


#### Randomize cloned disk and partitition UUIDs (GPT)

Randomize the disk and parition UUIDs
- Use with caution - filesystem data is not changed but you may have issues mounting drives, booting or assembling software raid devices

```
gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu

Command (? for help): x

Expert command (? for help): ?
a	set attributes
c	change partition GUID
d	display the sector alignment value
e	relocate backup data structures to the end of the disk
f	randomize disk and partition unique GUIDs
g	change disk GUID
h	recompute CHS values in protective/hybrid MBR
i	show detailed information on a partition
j	move the main partition table
l	set the sector alignment value
m	return to main menu
n	create a new protective MBR
o	print protective MBR data
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	resize partition table
t	transpose two partition table entries
u	replicate partition table on new device
v	verify disk
w	write table to disk and exit
z	zap (destroy) GPT data structures and exit
?	print this menu

Expert command (? for help): f
```

type w to write the changes to disk



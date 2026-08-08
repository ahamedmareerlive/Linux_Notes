# AWS EBS Root Volume Expansion on Ubuntu

## Complete Before-and-After Explanation

This guide explains how an AWS EBS root volume was expanded to **30 GB**, how Ubuntu detected the change, and why both the partition and filesystem had to be expanded.

---

## 1. Storage Layers

```text
AWS EBS volume
      ↓
Linux disk: /dev/nvme0n1
      ↓
Root partition: /dev/nvme0n1p1
      ↓
ext4 filesystem
      ↓
Mounted at: /
      ↓
Ubuntu, Terraform, packages and files
```

Increasing the EBS volume in AWS enlarges the virtual disk. Ubuntu must then expand:

1. The root partition
2. The filesystem inside the root partition

---

# Step 1: Check Available Filesystem Space

## Command

```bash
df -h
```

## Command Meaning

- `df` means **disk free**.
- `-h` means **human-readable**.
- It displays sizes using units such as MB and GB.

## Before Expansion

```text
Filesystem  Size  Used  Avail  Use%  Mounted on
/dev/root   6.7G  6.5G   181M   98%  /
```

## How to Read the Output

- **Filesystem:** `/dev/root`, the root filesystem
- **Size:** 6.7 GB total usable space
- **Used:** 6.5 GB already used
- **Avail:** only 181 MB free
- **Use%:** 98% full
- **Mounted on:** `/`, the main Ubuntu filesystem

## Why This Command Was Used

The command confirmed that the Ubuntu root filesystem was almost full:

```text
6.7 GB total
6.5 GB used
181 MB available
98% full
```

This caused Terraform and APT to fail with:

```text
No space left on device
```

---

# Step 2: Check Where the Terraform Project Is Stored

## Command

```bash
df -h ~/DevOps-Project/eks-install/
```

## Command Meaning

This checks which filesystem contains the Terraform project.

The `~` symbol means the current user's home directory:

```text
~ = /home/ubuntu
```

Therefore, the complete project path is:

```text
/home/ubuntu/DevOps-Project/eks-install/
```

## Output

```text
Filesystem  Size  Used  Avail  Use%  Mounted on
/dev/root   6.7G  6.5G   181M   98%  /
```

## What This Confirmed

The Terraform project was stored on the same nearly full root filesystem.

This command does **not** show how much space the project directory itself uses. To check the directory's actual size, use:

```bash
du -sh ~/DevOps-Project/eks-install/
```

## Difference Between `df` and `du`

```text
df = shows total, used and available space on a filesystem
du = shows space used by a particular file or directory
```

---

# Step 3: Check the Disk and Partition Sizes

## Command

```bash
lsblk
```

## Command Meaning

`lsblk` means **list block devices**.

It displays:

- Disks
- Partitions
- Device sizes
- Device types
- Mount locations

## Before Expanding the Partition

```text
NAME           SIZE  TYPE  MOUNTPOINTS
nvme0n1         30G  disk
├─nvme0n1p1    6.9G  part  /
├─nvme0n1p13  1023M  part  /boot
├─nvme0n1p14     4M  part
└─nvme0n1p15   106M  part  /boot/efi
```

## Device Names Explained

### `/dev/nvme0n1`

```text
Complete 30 GB EBS disk
```

AWS Nitro-based EC2 instances commonly show EBS volumes as NVMe devices.

### `/dev/nvme0n1p1`

```text
Partition 1, mounted at /
```

This is the main Ubuntu root partition. It contains most directories, including:

```text
/home
/var
/usr
/etc
/opt
```

### `/dev/nvme0n1p13`

```text
Boot partition, mounted at /boot
```

It stores files Ubuntu needs during startup, such as kernel and boot-related files.

### `/dev/nvme0n1p15`

```text
EFI system partition, mounted at /boot/efi
```

It contains files used by UEFI firmware to start Ubuntu.

### `/dev/nvme0n1p14`

```text
Small reserved partition
```

This partition should be left unchanged.

### `loop0`, `loop1`, and Other Loop Devices

Loop devices shown by `lsblk` are usually read-only Snap package images. They are normal and are not additional EBS volumes.

## The Problem Shown by `lsblk`

```text
Complete disk:  30 GB
Root partition: 6.9 GB
```

AWS had already expanded the complete EBS disk to 30 GB. However, partition 1 was still only 6.9 GB.

The remaining space was not yet assigned to the root partition:

```text
[Root partition: 6.9 GB][Unused space: about 22 GB]
```

---

# Step 4: Expand Root Partition 1

## Command

```bash
sudo growpart /dev/nvme0n1 1
```

## Command Breakdown

- `sudo` runs the command with administrator permission.
- `growpart` expands an existing partition into available unallocated space.
- `/dev/nvme0n1` is the complete EBS disk.
- `1` means partition number 1.

## Simple Meaning

```text
Expand partition 1 on the /dev/nvme0n1 disk.
```

## Before `growpart`

```text
Complete disk: 30 GB
Partition 1:    6.9 GB
Unused space:  about 22 GB
```

```text
[Partition 1: 6.9 GB][Unused space: about 22 GB]
```

## After `growpart`

```text
Complete disk: 30 GB
Partition 1:   28.9 GB
```

```text
[Partition 1: 28.9 GB]
```

## Command Output

```text
CHANGED: partition=1 start=2324480 old: size=14452703 end=16777182 new: size=60590047 end=62914526
```

## Output Explanation

- `partition=1` means partition 1 was expanded.
- `start=2324480` is the partition's starting location.
- `old size` is the original partition size in disk sectors.
- `old end` is the original ending location.
- `new size` is the expanded partition size.
- `new end` is the new ending location.

The starting position stayed the same. The ending position moved outward to use the available disk space.

`growpart` did not format the partition or delete files. It changed the partition boundary so partition 1 could use the unallocated space.

---

# Step 5: Confirm the Expanded Partition

## Command

```bash
lsblk
```

## Before `growpart`

```text
NAME          SIZE  TYPE  MOUNTPOINTS
nvme0n1        30G  disk
└─nvme0n1p1   6.9G  part  /
```

## After `growpart`

```text
NAME           SIZE  TYPE  MOUNTPOINTS
nvme0n1         30G  disk
└─nvme0n1p1   28.9G  part  /
```

## What Changed

```text
Root partition before: 6.9 GB
Root partition after:  28.9 GB
```

## What Had Not Changed Yet

The partition had expanded, but the filesystem inside it still needed to be expanded.

At this stage:

```text
Disk       = 30 GB
Partition  = 28.9 GB
Filesystem = still using its old size
```

A simple analogy is:

```text
Partition  = room
Filesystem = shelves inside the room
```

`growpart` made the room larger, but the shelves still covered only the original area.

---

# Step 6: Expand the Filesystem

## Command

```bash
sudo resize2fs /dev/nvme0n1p1
```

## Command Breakdown

- `sudo` runs the command with administrator permission.
- `resize2fs` resizes an ext2, ext3, or ext4 filesystem.
- `/dev/nvme0n1p1` is the root partition containing the filesystem.

## Simple Meaning

```text
Expand the filesystem inside root partition 1.
```

## Why This Command Was Required

`growpart` expanded the partition, but it did not automatically expand the filesystem inside it.

`resize2fs` allowed the ext4 filesystem to use all the newly available partition space.

## Before `resize2fs`

```text
Partition:  28.9 GB
Filesystem: about 6.7 GB usable
```

## After `resize2fs`

```text
Partition:  28.9 GB
Filesystem: about 28 GB usable
```

## Command Output

```text
Filesystem at /dev/nvme0n1p1 is mounted on /; on-line resizing required
```

This means:

- The filesystem was mounted at `/`.
- Ubuntu was actively using the filesystem.
- The filesystem could be expanded while the server was running.

This is called **online resizing**.

You did not need to:

- Stop the EC2 instance
- Unmount `/`
- Enter recovery mode
- Reboot Ubuntu

The final message was:

```text
The filesystem on /dev/nvme0n1p1 is now 7573755 (4k) blocks long.
```

This confirmed that the filesystem expansion completed successfully.

---

# Step 7: Confirm the Final Filesystem Space

## Command

```bash
df -h
```

## Before Expansion

```text
Filesystem  Size  Used  Avail  Use%  Mounted on
/dev/root   6.7G  6.5G   181M   98%  /
```

## After Expansion

```text
Filesystem  Size  Used  Avail  Use%  Mounted on
/dev/root    28G  6.5G    22G   23%  /
```

## Before Versus After

```text
Total usable space: 6.7 GB  → 28 GB
Used space:         6.5 GB  → 6.5 GB
Free space:         181 MB  → 22 GB
Disk usage:         98%     → 23%
Mount point:        /       → /
```

The existing data remained approximately 6.5 GB. The expansion added more available capacity without deleting existing files.

---

# Complete Command Flow

```bash
# 1. Check filesystem usage
df -h

# 2. Check which filesystem contains the Terraform project
df -h ~/DevOps-Project/eks-install/

# 3. Check the complete disk and partition sizes
lsblk

# 4. Expand partition 1 on the EBS disk
sudo growpart /dev/nvme0n1 1

# 5. Confirm that partition 1 expanded
lsblk

# 6. Expand the ext4 filesystem inside partition 1
sudo resize2fs /dev/nvme0n1p1

# 7. Confirm the new usable filesystem space
df -h
```

---

# Entire Process in One Visual

```text
BEFORE
======

AWS EBS disk
/dev/nvme0n1
30 GB
    │
    ├── Root partition /dev/nvme0n1p1: 6.9 GB
    │       └── Filesystem mounted at /: 6.7 GB
    │               ├── Used: 6.5 GB
    │               ├── Free: 181 MB
    │               └── Usage: 98%
    │
    └── About 22 GB not used by the root partition


COMMAND 1
=========

sudo growpart /dev/nvme0n1 1

Result:
Root partition increased from 6.9 GB to 28.9 GB.


MIDDLE
======

AWS EBS disk: 30 GB
    │
    └── Root partition: 28.9 GB
            └── Filesystem still needs expansion


COMMAND 2
=========

sudo resize2fs /dev/nvme0n1p1

Result:
The ext4 filesystem expanded to use the larger partition.


AFTER
=====

AWS EBS disk
/dev/nvme0n1
30 GB
    │
    └── Root partition /dev/nvme0n1p1: 28.9 GB
            └── Filesystem mounted at /: 28 GB
                    ├── Used: 6.5 GB
                    ├── Free: 22 GB
                    └── Usage: 23%
```

---

# Why 30 GB Appears as About 28 GB

The complete disk is 30 GB, but part of the disk is used by:

- `/boot`, approximately 1 GB
- `/boot/efi`, approximately 106 MB
- A small reserved partition
- Partition and filesystem metadata
- ext4 reserved capacity

Therefore:

```text
Complete EBS disk: 30 GB
Root partition:    28.9 GB
Usable root space: about 28 GB
```

This is normal.

---

# Was a Separate Mount Required?

No. This process expanded the existing root EBS volume.

The root partition was already mounted:

```text
/dev/nvme0n1p1 → /
```

Therefore, there was no need to:

- Run a separate `mount` command
- Create a new mount directory
- Add a new entry to `/etc/fstab`

Separate mounting would be needed only when attaching a new additional EBS volume, for example at `/data`.

---

# Important Safety Warning

Do **not** run the following command on the current root partition:

```bash
sudo mkfs.ext4 /dev/nvme0n1p1
```

`mkfs.ext4` creates a new filesystem. Running it on the root partition could erase Ubuntu, Terraform files, configuration files, and other data.

The root volume is already formatted, mounted, and expanded. No additional formatting is required.

---

# Useful Verification Commands

## Show the Root Filesystem Usage

```bash
df -h /
```

## Show Disks, Partitions, Filesystem Types and Mount Points

```bash
lsblk -f
```

## Show Partition Size and Filesystem Availability Together

```bash
lsblk -o NAME,SIZE,FSTYPE,FSAVAIL,FSUSE%,MOUNTPOINTS
```

## Confirm the Root Filesystem Type

```bash
findmnt -no SOURCE,FSTYPE,TARGET /
```

Expected output should be similar to:

```text
/dev/nvme0n1p1 ext4 /
```

## Check the Terraform Project Directory Size

```bash
du -sh ~/DevOps-Project/eks-install/
```

---

# Next Terraform Commands

After expanding the volume, retry Terraform:

```bash
cd ~/DevOps-Project/eks-install
terraform init -migrate-state
```

If the earlier failed download left incomplete local provider files, remove only Terraform's local working directory and initialize again:

```bash
rm -rf .terraform
terraform init -migrate-state
```

Then validate the Terraform configuration:

```bash
terraform validate
```

Do not delete your Terraform state file or the state stored in the S3 backend.

---

# Final Result

```text
EBS volume:       30 GB
Root partition:   28.9 GB
Root filesystem:  28 GB
Used:             6.5 GB
Available:        22 GB
Usage:            23%
Mounted at:       /
```

## Final Summary

```text
AWS increased EBS to 30 GB
        ↓
lsblk confirmed the disk was 30 GB
        ↓
growpart expanded partition 1
        ↓
resize2fs expanded the ext4 filesystem
        ↓
df -h confirmed 22 GB was available
```

The root EBS volume expansion is complete, persistent, and ready for Terraform.

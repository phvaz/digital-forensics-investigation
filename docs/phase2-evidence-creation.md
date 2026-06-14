# Phase 2 — Synthetic Evidence Creation

## Context

In real forensic investigations, the examiner works with a physical device
(HD, SSD, USB drive). In this controlled lab environment, a synthetic disk
image was created to simulate a suspect device, allowing the full forensic
workflow to be practiced without access to real hardware.

## What Was Done

A 200MB raw disk image was created using `dd`, partitioned with `parted`,
formatted as ext4, and mounted via loopback device. Files simulating a
fraud scenario were written to the partition, then intentionally deleted
to replicate a suspect attempting to destroy evidence.

## Commands Executed

```bash
# Create 200MB blank image
dd if=/dev/zero of=evidences/evidence.dd bs=1M count=200 status=progress

# Partition the image
parted evidences/evidence.dd mklabel msdos
parted evidences/evidence.dd mkpart primary ext4 1MiB 199MiB

# Mount via loopback and format
sudo losetup -Pf evidences/evidence.dd
sudo mkfs.ext4 /dev/loop0p1
sudo mount /dev/loop0p1 /mnt/forense

# Plant simulated evidence files
echo "PIX Transfer R$ 5000 CPF 123.456.789-00" > /mnt/forense/transfer_receipt.txt
echo "access_password: admin@2024" > /mnt/forense/credentials.txt
cp /usr/share/pixmaps/kali-menu.png /mnt/forense/meeting_photo.png

# Simulate evidence destruction
rm /mnt/forense/transfer_receipt.txt
rm /mnt/forense/credentials.txt

# Unmount
sudo umount /mnt/forense
sudo losetup -d /dev/loop0
```

## Key Concepts

**Why dd?**
`dd` performs a bit-for-bit copy of a device or file, producing a forensically
sound image that preserves every byte of the original — including deleted files,
unallocated space, and filesystem metadata.

**Why a VM?**
`dd` has no confirmation prompt. Swapping `if` and `of` overwrites the host disk
entirely. Running inside a VM isolates all operations from the host system.

**Why delete files?**
File deletion in Linux removes only the directory entry. The actual data remains
on disk until overwritten. This is the foundation of forensic file recovery —
demonstrated in Phase 4 (TSK) and Phase 5 (Foremost).

## Screenshots

- `screenshots/phase2_dd_creation.png` — dd creating the 200MB image
- `screenshots/phase2_parted.png` — partition table created with parted
- `screenshots/phase2_unmount.png` — image unmounted and loopback released

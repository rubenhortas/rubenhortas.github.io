---
title: Maintain NAND flash drives (SSD) in GNU/Linux
date: 2023-04-08 00:00:01 +0000
categories: [gnu/linux, administration]
tags: [gnu/linux, administration, hdd, nand, usb, ssd, fsck, flash drive]
---

NAND disk ranges from USB flash drives (thumb drive, memory stick, pendrive, call it what you prefer) to SDD (Solid State Drive) disks.
Both are based on [NAND flash](https://en.wikipedia.org/wiki/Flash_memory#NAND_flash) cells.

As hard disk drives (HDDs), NAND disks can suffer several types of failures: logical failures, firmware failures, and bit rot.
Since we cannot avoid firmware failures, we will try to avoid (or minimize) bit rot and logical failures.

# Bit rot

Transistors in the flash storage device hold charge which represents data.
This charge gradually leaks over time, leading to accumulation of logical errors (also known as "bit rot" or "bit fading").
The only thing we can do to avoid this is plug in periodically our disk or flash drives.
The paper, from 2015, ["Data Retention in MLC NAND Flash Memory: Characterization, Optimization, and Recovery"](https://people.inf.ethz.ch/omutlu/pub/flash-memory-data-retention_hpca15.pdf) from [Carnegie Mellon University](https://en.wikipedia.org/wiki/Carnegie_Mellon_University) states "*today's flash devices, which do not require flash refresh, have a typical retention age of 1 year at room temperature*".

# Logical failures

Flash memory has a finite number of P/E (program/erase) cycles.
Although many use a [wear leveling](https://en.wikipedia.org/wiki/Wear_leveling) strategy to work around these limitations, others don't.
Before a total disk death from usage occurs, some transistors can die from use causing data access errors.

# Maintenance

The maintenance process for a SSD disk is the same as in ["Maintenance of HDD in GNU/Linux"](https://rubenhortas.github.io/posts/maintenance-of-hdd-in-gnu-linux/).
But, the maintenance process differs a bit for an USB flash drive.

## Umount the disk

First of all is know the device assigned to the disk we want to check.
We can know the device assigned using `fdisk -l` or `lsblk`.
Let's say our disk is still `/dev/sdb`.

As the disk should be umounted to be able to run fsck, now, we need to umount it:

```
$ sudo umount /dev/sdb
```

## Run badblocks

In an USB flash drive we won't have S.M.A.R.T. capabilities, so we can skip this step.
Besides, as a flash drive is not a disk, a flash drive lacks of a reallocator that marks bad sectors and reallocates its data to spare sectors.
So, the best we can do is rely on badblocks. And, as we will use `e2fsck`, we will use `badblocks` through the `e2fsck -cc` option.

```
$ sudo e2fsck -fccky /dev/sdb1
```

Combining `-k` with `-cc` option, any existing bad blocks in the bad blocks list are preserved, and any new bad blocks found by running `badblocks` will be added to the existing bad blocks list.
Besides, with `-cc` option the bad block scan will be done using a non-destructive read-write test.

We could have also run badblocks directly:
```
$ sudo badblocks -sv /dev/sdb1
```

## Check badblocks results

To see the blocks marked as bad in a ext2/ext3/ext4 filesystem we have to use `dumpe2fs`:

```
$ sudo dumpe2fs -b /dev/sdb1
```

*Enjoy! ;)*

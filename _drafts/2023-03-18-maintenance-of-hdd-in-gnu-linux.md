---
title: Maintenance of HDD in GNU/Linux
date: 2023-03-18 00:00:01 +0000
categories: [gnu/linux, sysadmin]
tags: [gnu/linux, hdd, smartmontools, smartctl, fsck]
---

It's a good practice to check the health of your HDD (Hard Disk Drives) from time to time and repair them if neccesary.
It will avoid a lot of data loss and headaches.

The process can take anywhere from a few minutes to a few hours, but it's worth it.
Also, unless it is your main HDD, you can continue working while the disk is being checked and fixed.

How can we check and fix our HDD in GNU/linux?

# Preparing the disk

First of all is know the device assigned to the disk we want to check.
You can know the device assigned using `fdisk -l` or `lsblk`.
Let's say our disk is `/dev/sdb`.

> The disk should be umounted to be able to run smartctl
{: .prompt-warning}

# Check hard drive health using smartctl

`smartctl` is a utility contained on the `smartmontools` package.
`smartctl` serves for check the HDD [S.M.A.R.T. (Self-Monitoring Analysis and Reporting Technology)](https://en.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology) 
attributes and is the utility that we'll use for run some tests and check our HDD overall status.

## Check if SMART is enabled

```
$ sudo smartctl -i /dev/sdb | grep support 
``` 

Our output should be:

```
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

But, if our disk has SMART available and disabled, we should enable it with:

```
$ sudo smartctl -s on /dev/sdb
```

## Check the disk status

```
$ sudo smartctl -a /dev/sda
```

There's a lot of info displayed, but we should pay special attention to the next fields:
* Reallocated_Sector_Ct (Reallocated Sectors Count): The raw value represents a count of the bad sectors that have been found and remapped.
* Power_On_Hours: Count of hours in power-on state. It's not useful to check for errors, but it's useful to get an idea of the hours of life that the disk has left.
* Reported_Uncorrect: Reported Uncorrectable Errors. The count of errors that could not be recovered using hardware.
* Current_Pending_Sector: Count of "unstable" sectors (waiting to be remapped, because of unrecoverable read errors).

If the `RAW_VALUE` is greater than 0 for any of these fields, we should backup our files (if necessary) and we should fix the disk later.

## Estimate the test time

The `smartctl` utility can perform a variety of tests:

* **offline**: A short foreground test of less than two minutes. 
* **short**: Runs SMART Short Self Test (usually under ten minutes).
* **long**: A more accurate version of the “short” test. Could take a few hours.
* **conveyance**: Checks for possible damages occurred during the transportation of the device. Should take a few minutes.

And we can known the estimated duration of the tests executing:

```
$ sudo smartctl -c /dev/sdb
```

## Test the disk

I prefer to run the long test as it will give us a better overall disk health.

```
$ sudo smartctl -t long /dev/sdb
```

Once executed, `smartctl` will give us the neccesary time to complete the test:

```
Please wait 303 minutes for test to complete.
```

And, we can always cancel the test execution:

```
Use smartctl -X to abort test.
```

After the time specified by `smartctl` we can check the test results:

```
$ sudo smartctl -a /dev/sdb
```

# Fix the filesystem using fsck

`fsck` (File System Consistency Check) comes by default on GNU/Linux distributions.
`fsck` is used to check to check and, optionally, repair one or more Linux filesystems.

> The disk should be umounted to be able to run fsck
{: .prompt-warning}

## Check the partitions

Let's say we want repair our `/dev/sdb1` partition.

Sometimes the disk is marked as clean, but we know for sure that the disk has some damage, because we had errors using it.
So, we can force a check on the partition:

```
$ sudo fsck -f /dev/sdb1
```

Don't worry, this test is fast ;)

## Fix the filesystem automatically

The most confortable wat to repair the this is do it in "autopilot mode" or automatically.
We can do this in two ways:

* Automatic repair (no questions):
  ```
  $ sudo fsck -p /dev/sdb1
  ```
* Assume "yes" to all questions:
  ```
  $ sudo fsck -y /dev/sdb1
  ```
  
*Enjoy! ;)*

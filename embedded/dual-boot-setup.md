# Dual-Boot


## Definition
Dual boot is a technology that allows user to install two or more operating systems on one device.

For example it allows to install Windows and Linux distro on one machine to get best of two OS.

## Why?

One usage example is to speed-up compiling times.

Due to differences in File System implementation in Windows ([NTFS](https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview)) and Linux([ext4](https://docs.kernel.org/admin-guide/ext4.html)), compile time for large projects/files can differ.

Here are compile times for the same project on Windows and Linux:

![Windows compile time](./images/windows_build_time.png)
> Windows compile time


![Linux compile time](./images/linux_build_time.png)
> Linux compile time

Other usage may be related to obscense of required software on used OS.

## How?

There are two ways of setting up dual boot:
* Install second OS on a separate drive
* Divide existing drive in two parts and install OS there (not recommended in terms of security and safety)

This instruction will cover the first option and explain in details steps to install Windows and Linux OS on separate hard drives with encryption enabled.

Also this instruction covers the case where your main OS is Windows and you want to install Linux as a second OS.

### Prerequisites

* Installed Windows with Bitlocker enabled (*optional, bitlocker may be enabled/installed afterwards)
* Empty SSD
* Paper, pen to write down decryption key
* Fresh Windows boot image
* Fresh Ubuntu boot image
* Boot USB drive

### Steps

Create a boot drive with Ubuntu OS.

> TO-DO: discussion about creating boot drive here?

> TO-DO: discuss if instruction for bitlocker setup is required here?

Set-up bitlocker normally.  
Enable Secure    Boot if required.

> If bitlocker was enabled beforehand, remove Windows hard drive

Change boot order in BIOS/UEFI.

Load into Ubuntu setup.

Set-up encryption.

> TO-DO: add screenshots, images

Select preferred encryption algo.

Write down encryption key.

**Ubuntu setup is finished!**

Set up boot prioritty to new Ubuntu SSD as it will force laptop to load into GRUB bootloader where you can easily select system to load without possibly triggering Bitlocker

If we try to load Windows with enabled bitlocker we may be  faced with bitlocker warning every system load.

To avoid it we have to perform additional steps.

* Select Windows as a 1st priority boot device

Alternatively you may set up bitlocker on Windows after Linux installation and there will be no bitlocker warning triggered :relieved:
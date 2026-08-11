# Dual-Boot


## Definition
Dual booting refers to the process of installing and running two different operating systems on a single computer. This allows you to choose between the two when you start your computer, giving you the flexibility to switch between them based on your needs [[1](#1-what-is-dual-boot)].

## Why?

One usage example is to speed-up compiling times.

Due to differences in File System implementation in Windows (NTFS [[2](#2-ntfs-overview)]) and Linux(ext4 [[3](#3-ext4-general-information)]), compile time for large projects/files can differ.

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

* Windows Installed on system
  * Option 1: Enabled Bitlocker
  * Option 2: Clean Windows
* Empty SSD
* Paper, pen to write down decryption key
* Fresh Windows boot image
* Fresh Ubuntu boot image
* Boot USB drive

### Steps

Create a boot drive with Ubuntu 22.04 or 24.04.

> If Windows was prepared with Option 1 - disable bitlocker before installing Linux

Disable Secure Boot before installing Linux.

> If you have not disabled Bitlocker on Windows or can't do it because of other reasons please try to remove Windows SSD. Otherwise you will not be able to avoid recovery key trigger on each Windows load or even won't be allowed to start system. 

Change boot order in BIOS/UEFI and select USB drive as 1st priority.

Load into Ubuntu setup.

During disk setup:
* Select `Erase Disk and install Ubuntu`
* Click `Advanced Features`
* Select `Use LVM and encryption`
    * Encryption used here is `Linux Unified Key Setup`
* Select new drive or create partitions

Set-up encryption:
* Create a passphrase (you have to enter it every time on login screen)
* Save recovery key (if you've installed Ubuntu 22.04)

Set-up user account.

**Ubuntu setup is finished!**

Set up boot prioritty to new Ubuntu SSD as it will force laptop to load into GRUB bootloader where you can easily select system to load without possibly triggering Bitlocker.

> If Bitlocker was enabled on Windows before Linux instalation it will trigger each time we select Windows manually in GRUB. To avoid it set Windows SSD as first boot device and load Linux from BIOS/UEFI.

> If Bitlocker was disabled before Linux installation, please enable it normally and encrypt entire drive. Save recovery key on separate device or print immediately.

#### Possible problems

* If you are using Dell laptop, please select Bitlocker Audit mode in BIOS/UEFI settings.


# References
### 1. What is dual boot?
 https://www.lenovo.com/us/en/glossary/what-is-dual-boot/?orgRef=https%253A%252F%252Fwww.google.com%252F&srsltid=AfmBOoomzAuc0sCTISqrZOgOO7qszHY_1z4p5ih9I-KQMG4H6kOtb7z-
### 2. NTFS Overview
https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview
### 3. ext4 General Information
https://docs.kernel.org/admin-guide/ext4.html
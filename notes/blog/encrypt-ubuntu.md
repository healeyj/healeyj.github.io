title: Encrypt Ubuntu Across Two Disks (SSD and HDD)
description: In this tutorial, we'll setup an encrypted install of Ubuntu 20.04 LTS across two disks.
tags: linux
tags: security
published: 2021-02-25
modified: 2021-02-25
template: blog
length: 10
related: 
---

## INTRODUCTION

In this tutorial, we'll setup an encrypted install of Ubuntu 20.04 LTS on a small SSD (our primary drive). Then, We'll make a second encrypted partition on a larger HDD (our secondary drive). Finally, we'll link these two encrypted disks together with some command-line sorcery (block device mapping) and move our lovely `/home` folder to that secondary drive.

## ACKNOWLEDGEMENTS

This post is written mostly to testify that the following tutorial works perfectly on the latest version of Ubuntu LTS - 20.04 at the time of writing.

[Ubuntu – How to install 18.04 using full disk encryption with two drives (SSD/HDD)](https://itectec.com/ubuntu/ubuntu-how-to-install-18-04-using-full-disk-encryption-with-two-drives-ssd-hdd/).

I initially followed the above tutorial and completed it with Ubuntu 18.04 LTS Bionic Beaver. Still, I wondered, "maybe this'll work with Ubuntu 20.04 LTS Focal Fossa. Why not give it a shot?" I did, and again, it works flawlessly with the latest version of Ubuntu.

I'll try to compress some parts of the [iTecTec tutorial](https://itectec.com/ubuntu/ubuntu-how-to-install-18-04-using-full-disk-encryption-with-two-drives-ssd-hdd/) and expand upon others.

---
## OVERVIEW

### concepts

A basic understanding of the following concepts will help but I'll do my best to explain some of the more difficult ones as we go along.

* drives
* partitions
* pathing
* mounting
* file systems
* block devices
* encryption
* keyfiles

### command-line toolbox

This is where the magic happens. If you're familiar with these tools already, you can complete this whole process in less than an hour.

* Ubuntu default Disk application (gparted is OK too)
* sudo
* cryptsetup
* mkfs
* dd
* chmod
* a text editor (nano, vim, or a GUI)
* blkid
* mkdir
* mount
* rsync

---
## PART I
### Install Ubuntu 20.04 LTS on the primary drive and then encrypt a partition on the secondary drive

First and foremost, do a fresh install of Ubuntu 20.04 LTS on the SSD (primary drive). To create a USB installer, see my recent blog post about creating a bootable USB with dd.

In the installation wizard, we must check the boxes "Encrypt the new Ubuntu installation for security" and "Use LVM (Logical Volume Management) with the new Ubuntu installation".

The installer will prompt us to create a secuirty key. This is a password that we will use to unlock our SSD everytime we boot into Ubuntu. It is *distinct* and preferably different from our user account password.

---

Secondly, use `cryptsetup` to setup a partition on the secondary drive as an encrypted [LUKS](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) (Linux Unified Key Setup) partition. The [cryptsetup](https://reposcope.com/package/cryptsetup) package is installed by default in most popular Linux distros.

Replace `?X` with the appropriate identifiers of the target partition. The letter that replaces "?" refers to the drive and the number that replaces "X" refers to the partition on that drive.

You'll need to set a new password for this LUKS partition. Again, this is distinct from both your user account password and your primary drive password.

For recap, we've now set up three (3) distinct passwords, one for the encrypted primary drive, one for your user account, and one for the LUKS-encrypted partition on the secondary drive.

`sudo cryptsetup -y -v luksFormat /dev/sd?X`

---

Next, we'll use `cryptsetup` again. `luksOpen` will decrypt the secondary drive you just created.

The `mkfs` (make file system) command run with the `.ext4` (fourth extended filesytem) parameter will format the *unencrypted* LUKS partition as ext4. Essentially ext4 is a Linux compatible filesystem, like [NTFS](https://en.wikipedia.org/wiki/NTFS) is for Windows or [APFS](https://en.wikipedia.org/wiki/Apple_File_System) is for macOS. By wrapping our ext4 filesystem with that LUKS partition, we ensure that our filesystem and *all* of its data will be encrypted.

Note that `cryptsetup` is an interface to mount this encrypted partition with a mapper, which is essentially a symbolic link. The Wikipedia article for [Device Mapper](https://en.wikipedia.org/wiki/Device_mapper) explains that it "works by passing data from a virtual block device... to another block device. Data can be also modified in transition, which is performed, for example, in the case of device mapper providing disk encryption".

Briefly, the mapper affords us the luxury of encryption.

```
sudo cryptsetup luksOpen /dev/sd?X sd?X_crypt
sudo mkfs.ext4 /dev/mapper/sd?X_crypt
```

---

Next, we will create a keyfile that will serve as our personal key to unlock that encrypted LUKS parition. We'll store the key on the primary partition and set it up so that when we login, the keyfile will automatically unencrypt that secondary drive.

We'll use `dd` to pull noise from our system's random number generator, [`/dev/urandom`](https://en.wikipedia.org/wiki//dev/random) and make a pseudo-random and strong password at `/root/.keyfile`.

With `chmod`, we'll give our user account (read-access)[https://chmodcommand.com/chmod-0400/] to the keyfile.

Then coming back to the wonderful `cryptsetup`, we'll use `luksAddKey` to link our keyfile to that encrypted secondary partition.

Note that this keyfile is distinct from the three (3) passwords we are working with.

```
sudo dd if=/dev/urandom of=/root/.keyfile bs=1024 count=4
sudo chmod 0400 /root/.keyfile
sudo cryptsetup luksAddKey /dev/sd?X /root/.keyfile
```

---

Run `blkid` and copy the `UUID` of the `sd?X` filesystem. Note that this is different from the already decrypted `sd?X_crypt`. Nor do we want the `PARTUUID`.

What does `blkid` mean though? A quick Stack Exchange search shows ("Block (device) identification")[https://unix.stackexchange.com/questions/212866/what-does-blkid-stand-for]. From Wikipedia, ["block devices provide buffered access to hardware devices, and provide some abstraction from their specifics"](https://en.wikipedia.org/wiki/Device_file#Block_devices). UUID means "Universal Unique Identifier", it allows use to uniquely identify the parition we're working with.

Essentially, block devices make it easy for users like us to work with filesystems and do fun stuff like encrypt them and link them across drives.

`sudo blkid`

---

With our target UUID copied to the clipboard, open up `/etc/crypttab` with a text editor of choice. Nano, vim, emacs, or a GUI text editor will work.

`sudo nano /etc/crypttab`

Copy and paste the following line to the last line of the text file:

`sd?X_crypt UUID=<device uuid> /root/.keyfile luks,discard`

Note that we are taking the UUID of `sd?X` from `blkid`, linking it to the keyfile that we've set up to unlock it, and telling our system to reference it as `sd?X_crypt`.

---

All done! We've successfully linked partitions across our SSD and HDD, and both should automatically unlock upon login. Test this by restarting and then moving some files back and forth between the two drives.

To wrap up, let's move the `/home` folder to our secondary drive (HDD) to reserve space on our primary drive (SSD) for the operating system we installed.

---
## PART II
### Part II: Move the /home folder to the secondary drive

First, create a new temporary directory at `/mnt/tmp`, then mount `/dev/mapper/sd?X_crypt` to it.

If there's ever doubt as to what a certain command does, use `whatis` (thanks again to [that stackexchange post on blkid](https://unix.stackexchange.com/questions/212866/what-does-blkid-stand-for)) For example, try running `whatis mkdir` in the terminal.

```
sudo mkdir /mnt/tmp
sudo mount /dev/mapper/sd?X_crypt /mnt/tmp
```

---

Next, we'll use `rsync` to copy our `/home` folder and all its contents to our new temporary mount point on the second drive.

The `rsync` command is an improvement on `rcp`, remote copy, which functions about the same as `cp`, which simply copies. We need a *remote* copy command because we're working across drives.

`sudo rsync -avx /home/ /mnt/tmp`

---

Let's mount our `/home` folder onto our unlocked secondary partition.

`sudo mount /dev/mapper/sd?X_crypt /home`

---

Almost done.

Open up `/etc/fstab` with a text editor:

`sudo nano /etc/fstab`

Now copy and paste the following line to the end of the text file:

`/dev/mapper/sd?X_crypt /home ext4 defaults 0 2`

What is `fstab`?

It's an important system file that stores "static information about the filesystems" (from `whatis fstab`). The Wikipedia entry on [fstab](https://en.wikipedia.org/wiki/Fstab) says that the `mount` command relies on it to function. This is logical since filesystems exist to keep our files organized, and `mount` helps us access devices outside our primary filesystem. 

By permenantly writing our secondary partition and `/home` folder paths to fstab, we ensure that our operating system will recognize that partition as the new *home* of our `/home` folder.

---

## CONCLUSION

If you've made it this far, pop a bottle of champagne.

First, we did an encrypted install of Ubuntu 20.04 LTS on our primary drive. We then set up an encrypted LUKS partition on a secondary drive tied to a randomly-generated keyfile that triggers on login. Finally, we used some black magic (block device mapping) to permenantly move our home folder onto it.

Since we're running the latest version of Ubuntu LTS, we'll enjoy [public security maintenance into 2024, and Extended Security Maintenance until 2030](https://ubuntu.com/about/release-cycle).

Big thanks again to iTecTec for the [original tutorial](https://itectec.com/ubuntu/ubuntu-how-to-install-18-04-using-full-disk-encryption-with-two-drives-ssd-hdd/).
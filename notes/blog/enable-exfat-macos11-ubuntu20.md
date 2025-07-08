title: Enable exFAT on macOS 11.2 and Ubuntu 20.04
description: In this tutorial we will set up enable exFAT compatibility on macOS 11.2 and Ubuntu 20.04.
tags: linux
tags: macos
published: 2021-03-06
modified: 2021-03-06
template: blog
length: 2
related: 
---

## Introduction

In this tutorial we will set up enable exFAT compatibility on macOS 11.2 and Ubuntu 20.04.

There are 2 advantages of the [exFAT](https://en.wikipedia.org/wiki/ExFAT) (Extensible File Allocation Table) filesystem: (1) it allows storage of files greater than four gigabytes, and (2) it is compatible across Windows, macOS, and Linux. If you want to move large files to and from all of these operating systems, exFAT is a solution.

Unfortunately, neither macOS 11.2 nor Ubuntu 20.04 include exFAT compatibility by default. We will install the necessary tools manually.

Note: Windows 10 supports exFAT by default, probably because Microsoft created the filesystem.

---

## macOS 11.2 - enable exFAT compatibility

We'll use [osx-fuse](https://osxfuse.github.io/) to enable exFAT compatibility with macOS Big Sur.

```
sudo brew update
sudo brew upgrade
sudo brew install osx-fuse
```

To finish the installation, you will need to restart your computer.

---

## Ubuntu 20.04 - enable exFAT compatibility

We'll use exfat-utils to enable exFAT compatibility in Ubuntu Focal Fossa.

```
sudo apt update
sudo apt upgrade
sudo apt install exfat-utils
```

---

## Conclusion

That's it. We set up `osx-fuse` and `exfat-utils` on macOS and Ubuntu respectively. Now we can create exFAT filesystems that will accomodate large files across operating systems.

### Sources / Additional Reading

* [Using exFAT on OSX](http://tech.saigonist.com/b/osx/using-exfat-mac-osx.html) by Tomo Saigon, 2016

* [exFAT Compatibility in Ubuntu](https://linuxhint.com/exfat_compatibility_ubuntu/) by Ranvir Singh, 2019

* [How to Format a USB Disk as exFAT on Linux](https://itsfoss.com/format-exfat-linux/) by Dimitrios, 2020



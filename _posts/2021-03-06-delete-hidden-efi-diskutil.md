---
layout: post
title:  "Delete hidden EFI partitions with macOS 11.2 diskutil"
date:   2021-03-06 11:00:00 +0000
categories:
---

# Delete hidden EFI partitions with macOS 11.2 diskutil

## Introduction

The Disk Utility GUI application included with macOS 11 hides some EFI partitions to protect users from damaging certain disks.

Let's use `diskutil` to find and delete an EFI partition that is hidden from view in the Disk Utility GUI application.

---

## Identify the partition

Find the identifier of the EFI parition you need to delete by running the following command.

{% highlight shell %}
`diskutil list`
{% endhighlight %}

---

## Delete the partition

Once you have identified the EFI parition that you need to delete, run the following command. Be sure to replace "X" and "Y" in "diskXsY" with your target disk number and partition number, respectively.

{% highlight shell %}
`diskutil eraseVolume FREE UNTITLED diskXsY`
{% endhighlight %}

---

## Conclusion

That's it! We deleted an EFI partition from a disk with the `diskutil` tool. 


Bonus: Remember, `man` is your friend. Use "j" and "k" to scroll up and down one line at a time with `less` (the text file viewer used with `man`). Additionally, you can use control + f and control + b to scroll down and up one page at a time.
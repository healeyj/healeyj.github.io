title: Make a bootable USB with dd on Linux and macOS
description: you've probably used uNetBootin, Etcher, and Rufus a dozen times to make a bootable usb drive. your grandma loves those apps.
tags: linux
tags: macos
published: 2021-02-25
modified: 2021-02-25
template: blog
length: 5
related: 
---

## INTRODUCTION

you've probably used [uNetBootin](https://href.li/?https://unetbootin.org), [Etcher](https://href.li/?https://www.balena.io/etcher/), and [Rufus](https://rufus.ie/) a dozen times to make a bootable usb drive.

your grandma loves those apps. they do all the work for you. but they're boring and you live dangerously.

enter the terminal. what could possibly go wrong? well, total corruption of the usb stick and/or your computer's operating system. don't worry though, a little caution goes a long way.

credit where it's due: this post is an expansion of [this askubuntu thread](https://href.li/?https://askubuntu.com/questions/372607/how-to-create-bootable-ubuntu-usb-flash-drive-from-terminal).

i'll explain step-by-step how the dd terminal command referenced in the link above works. if you're on macOS there is a small variation that i will reiterate. note that this won't work on windows.

---

## make a bootable usb in the terminal with dd

`sudo dd bs=4M if=path/to/input.iso of=/dev/sd conv=fdatasync status=progress`


to run this commmand, you would replace `path/to/input.iso` with the path to your linux .iso, and `/dev/sd>` with the path to your flash drive.

"but wait", you say, "what the hell is going on here?"


great question. let's take a closer look.

---

`sudo`


god of the terminal. short for "superuser do", it runs the following command as an administrator, so you can do _fun_ stuff like break everything or make a bootable usb. we'll do the latter. when you run sudo for the first time you'll be prompted for your password.

--- 

`dd`

some ibm sorcery, apparantly. without getting too bogged down with this proprietary witchcraft, `dd` is a lot like `cp`, an easier-to-swallow terminal command that **c**o**p**ies a file from one location to another.

note that all of the bits that follow dd are its arguments or parameters. please excuse the jargon, they're the pieces of information that dd needs to know what to copy and where to copy it.

---

`bs=4M`

specifies the amount of data we will write at a time. it means, "block size equals 4 megabytes". so we will write **4** **M**egabytes of data at some interval. it doesn't really matter what that interval is, we just want a damn live usb.

### attention macOS users!

instead of `bs=4M` make the M lowercase: `bs=4m`. why? it's not the computers that suck, it's the people who make them.

---

`if=path/to/input.iso`

`if` means "input file"! it took me forever to figure that out because of all the if-statement torture i'm conditioned to. `path/to/input.iso` is the path to your linux .iso. further explanation is out of the scope of this post.

---

`of=/dev/sd`

`of` means "output file", duh. this is the part where your computer explodes. no, really. the path to your target usb goes here. replace with the drive letter. triple-check it, then quadruple-check it. that being said, failure is learning. again, identifying that drive letter is out of the scope of this post.

---

`conv=fdatasync`

"conversion equals file data sync". this is where the magic's at. what is file data sync? the man page (manual) says, "physically write output file data before finishing". [this stackexchange thread](https://href.li/?https://unix.stackexchange.com/questions/312687/why-is-sync-so-important-when-making-a-bootable-linux-usb-stick) explains that "sync" ensures the data is fully copied. again, out of scope, and who cares about that nerd stuff?! we're practically done!

---

`status=progress`

finally something that makes sense. this piece ensures we see some feedback in the terminal while the .iso copies to the usb. it'll show a cute lil' progress bar to reassure impatient users like ourselves.

---

## CONCLUSION

we just broke down every piece of the following command.

`sudo dd bs=4M if=path/to/input.iso of=/dev/sd conv=fdatasync status=progress`

now fill out your paths and burn those usbs to the ground.

the terminal is way faster, cooler, and more dangerous than uNetBootin and Etcher. cool grandmas dd.
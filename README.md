# dropbear-init-fix

A script to fix the inconsistency found in *dropbear-initramfs* package in *Ubuntu 18.04.x* .

## Overview

This is part of the effort that I've made to make remote unlocking of a LUKS encrypted server, work seamlessly on *Ubuntu 18.04.x* .

If you haven't read the related blog post yet, Its time to do so:

[How to install LUKS encrypted Ubuntu 18.04.x Server and enable remote unlocking](https://hamy.io/post/0009/how-to-install-luks-encrypted-ubuntu-18.04.x-server-and-enable-remote-unlocking/)


## Background

At one point in the past, the developers decided to reduce the size of *busybox-initramfs* package by using the stripped down version of busybox (Which in return, would have reduced the size of initramfs as a whole).

This however, started breaking couple of scripts here and there which relied on the full busybox version. While the developers have fixed most of these issues since, some changes have not been backported to *Ubuntu 18.04.x* yet.

As the result, there is a problem when you try to remote unlock a LUKS encrypted server running *Ubuntu 18.04.x*.

This problem manifests itself in these forms:

* Remote SSH sessions might not get closed automatically after a successful remote LUKS unlocking.
* You will get a couple of `ps` errors in your terminal right after LUKS unlocking.
* You might get this scary message in your terminal after boot up:  
  > Aiee, segfault! You should probably report this as a bug to the developer
* Your network interfaces *might* refuse to automatically come up again

## How it works

By backporting the required changes to the affected *dropbear-initramfs* script, this workaround aims to fix what's broken. 

It uses a non-aggressive approach to apply the fix right when initramfs is being generated. I've written it in a way to ensure that it would have no ill effect on your system. In a nutshell:

* It will not make any permanent changes to your system (only a file inside initramfs will be changed).
* It detects future changes to the affected package and will not apply the patch if it's not compatible. So worse comes worst, you will just encounter the original bug again, nothing more (more likely than not though, a future upgrade to *dropbear-initramfs* will fix this bug anyways).
* The approach taken in this workaround, is the same one as the newer builds of *dropbear-initramfs*. So this is basically a backport of the changes, without permanently adjusting any system files.
* If you ever decide to remove the workaround, you just need to remove the file and rebuild initramfs, that's it.

## Installing

The required steps are really simple:

* Change to *initramfs-tools* hooks directory:
  * `cd /etc/initramfs-tools/hooks`

* Download the workaround script:
  * `sudo wget https://raw.githubusercontent.com/HRomie/dropbear-init-fix/master/zz-dropbear-init-fix`

* Give the script +x permission:
  * `sudo chmod +x zz-dropbear-init-fix`

* Rebuild initramfs:
  * `sudo update-initramfs -u`

## Uninstalling

If for whatever reason you decided to remove the workaround and revert back to the original state, you would do the followings:

* Remove the script:
  * `sudo rm /etc/initramfs-tools/hooks/zz-dropbear-init-fix`

* Rebuild initramfs:
  * `sudo update-initramfs -u`

## Troubleshooting

Each time you rebuild initramfs, the workaround either gives you a success message indicating that it's doing its job, or gives out a very clear warning indicating why it couldn't apply the patch.

If you get a warning, the patch will not be applied (it would be as if the workaround is not there at all).

If you believe that this workaround should apply to you and yet you are getting a warning message, please share that with me in the [comments section of the related blog post](https://hamy.io/post/0009/how-to-install-luks-encrypted-ubuntu-18.04.x-server-and-enable-remote-unlocking/#disqus_thread).



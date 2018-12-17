# Android Development with 32bit Linux (Debian)

Google removed support for 32bit Linux for there build-tools for Android.
Last officially supported version is `23.0.3`. This version is too old for current development.

The main problem is that aapt2 is a 64bit executable and won't run on 32bit.

Lately I found a solution for this problem.
You can emulate a 64bit system for exactly one executable with [qemu](https://www.qemu.org/).  
But when just running it you will see the missing libraries. 

Those are in `/lib/x86_64-linux-gnu`.

## Setup

Obviously these libraries won't be found on a 32bit system because they are for 64bit.
This means installing them with apt will be difficult (at least I don't know how).

- You will need [libc6](https://packages.debian.org/stretch/amd64/libc6), [libc6-dev](https://packages.debian.org/stretch/amd64/libc6-dev), [zlib1g](https://packages.debian.org/stretch/amd64/zlib1g) and [libgcc1](https://packages.debian.org/stretch/amd64/libgcc1).
You should download the deb packages for your system (64bit instead of 32bit) into a new working folder.
- Create a folder `fakeInstall` in this folder.
- Now run:

  ```bash
  dpkg -x libc6_2.24-11+deb9u3_amd64.deb fakeInstall
  dpkg -x libc6-dev_2.24-11+deb9u3_amd64.deb fakeInstall
  dpkg -x zlib1g_1.2.8.dfsg-5_amd64.deb fakeInstall
  dpkg -x libgcc1_6.3.0-18+deb9u1_amd64.deb fakeInstall
  ``` 
  (update the versions to yours)
  
  This will install the packages into the `fakeInstall` folder.
- Now we need to put everything into the right place:  
  (attention when working with `sudo`!)
  
  ```bash
  sudo mv fakeInstall/lib/x86_64-linux-gnu /lib/
  sudo mkdir -p /lib64
  sudo mv fakeInstall/lib64/ld-linux-x86-64.so.2 /lib64
  ```
- Basically the preparation is finished.  
  So let's install [qemu](https://www.qemu.org/). Luckily it's in the default apt list  
  (at least on debian `stretch`):
  
  ```bash
  sudo apt-get update
  sudo apt-get install qemu qemu-user
  ```
- Last thing to do is to use `qemu` with `aapt2` instead of only `aapt2`

  ```bash
  mv $ANDROID_HOME/build-tools/28.0.1/aapt2 $ANDROID_HOME/build-tools/28.0.1/_aapt2
  ```
  Now move the aapt2 script from this repo into `$ANDRID_HOME/build-tools/28.0.1/` and give it execution permission by
  ```bash
  chmod +x $ANDROID_HOME/build-tools/28.0.1/aapt2
  ```
  
  If you don't have a global ANDROID_HOME variable replace it with the corresponding path.  
  Replace the build-tools version with yours!
  
  At some installations / in newer versions you also need to replace the `aapt2` in
  `~/.gradle/caches/transforms-*/files-*/aapt2-*-linux.jar/*/aapt2-*-linux/` like above.
  This is a cache which means you need to try to build it and replace it afterwards.  
  The `*` needs to be replaced with your specific versions (there should be only one at first).
  
- That's it. You can now build on 32bit Linux apps for Android.  
  Your working folder can be deleted. There is no reference to it.

## Further Information
I have tested this:  
```
$ uname -a
Linux Generic 4.9.0-7-686-pae #1 SMP Debian 4.9.110-1 (2018-07-05) i686 GNU/Linux

$ qemu-x86_64 -version
qemu-x86_64 version 2.8.1(Debian 1:2.8+dfsg-6+deb9u4)
Copyright (c) 2003-2016 Fabrice Bellard and the QEMU Project developers
```
Maybe it also works on other distro than Debian.  
Currently I can't test more than pure Debian.

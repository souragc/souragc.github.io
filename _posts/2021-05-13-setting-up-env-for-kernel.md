---
layout: post
title: Debugging a kernel challenge.
date: 2021-05-13 17:47:44
summary: How to debug a kernel challenge in a ctf.
tags:
 - kernel
 - ctf
 - debugging
---

This post is created so that I can come back if I forget how to do any of these. 
Also someone might find it useful someday.

These are the steps that I take when I get a kernel challenge during a ctf. I will
be using the challenge `Sharedhouse` from ASIS 2020 Quals. Pretty much the same steps
apply for any other challenge.

### Given Files

Let's see what all files are provided to us.
- bzImage: Compressed kernel linux kernel. We can extract it using [this](https://raw.githubusercontent.com/torvalds/linux/master/scripts/extract-vmlinux).
- rootfs.cpio: This is used as the file system by the kernel. The vulnmod is likely to be inside it.  
You can extract it using the following:

```sh
$ file rootfs.cpio
rootfs.cpio: ASCII cpio archive (SVR4 with no CRC)

$ mkdir rootfs && cd rootfs
$ cat ../rootfs.cpio | cpio --extract

```
- start.sh: Script that runs qemu with the proper parameters.

```sh
#!/bin/sh
qemu-system-x86_64 \
    -m 256M \
    -kernel ./bzImage \
    -initrd ./rootfs.cpio \
    -append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 kaslr pti=off quiet" \
    -cpu qemu64,+smep \
    -monitor /dev/null \
    -nographic


```
We can see that only kaslr and smep are enabled and smap and kpti are disabled.

### rootfs

If we check inside the rootfs, we can see that the module named `note.ko` is given.
Another file we have to looks at is the `init` file. This is a script that changes
the permission of files and then inserts the module. 
First thing I does is remove the folllowing lines from the `init`. It might
not be the same for other challenge.

```sh
echo 1 > /proc/sys/kernel/kptr_restrict
echo 1 > /proc/sys/kernel/dmesg_restrict
```

This will help later when we want to view `dmesg` and `/proc/kallsyms`. Another thing 
to change is the `start.sh`. Change the `kaslr` to `nokaslr` and add `-s` as another 
parameter. First one disables kaslr which will be really useful when we want to debug 
and the `-s` option creates a gdb server at `localhost:1234` which we will use to debug
the module. For ease of use, we can compile our exploit and put it in the rootfs directory,
compress it and use.

```sh
$ cd rootfs
$ find . | cpio -o -H newc > ../rootfs.cpio
```

### GDB

Extract the kernel using the `extract.sh`

```sh
./extract.sh ./bzImage > vmlinux
```

Now we can open it in gdb. Before doing that, run the `start.sh`.

```sh
gdb ./vmlinux
```

Now we connect to the server using `target remote: 1234`.
If we want to set breakpoints in the module, we have to add it in gdb.
To do this, we need the base address of the module. 

```sh
/ # cat /proc/modules 
note 16384 0 - Live 0xffffffffc0000000 (O)
/ # 

```

After this, run the following from gdb.

```sh
add-symbol-file note.ko 0xffffffffc0000000
```

Sometimes we might need address of some functions. To get this, we can use the `/proc/kallsyms` and
`grep` it's contents. Also it is better to do everything while you are root and then change it
once your exploit is finished. To change this, change the `id` set by `init` by editing the `init`.

### Conclusion

This post was about how to start debugging a kernel challenge in a ctf. Exploitation of the challenge
will depends on the challenge. But the initial steps should be similar to what I have said.

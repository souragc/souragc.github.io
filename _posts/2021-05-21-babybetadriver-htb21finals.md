---
layout: post
title: Baby beta driver - HTB 2021 Finals
date: 2021-05-21 18:41:06
summary: Writeup of the challenge baby beta driver from htb 2021 finals.
tags:
 - ctf
 - kernel
---

### Initial analysis

We are given the following files:
- bzImage
- initramfs.cpio
- run_challenge.sh

### RE

The module registers a character device `baby_beta_driver`. We are given two operations using ioctl: Store and Read.  

This is the ioctl code:
```c
mutex_lock(&safety_lock);
  copy_from_user(&req, arg, 16LL);
  if ( cmd == 0x1337C0DE )
  {
    beta_storage.size = req.size;
    if ( beta_storage.data )
      kfree();
    v5 = _kmalloc(beta_storage.size, 3264LL);
    beta_storage.data = v5;
    if ( req.transfer_data && beta_storage.size <= 0x7FFFFFFFuLL )
      copy_from_user(v5, req.transfer_data, beta_storage.size);
    printk(&unk_290);
    goto LABEL_10;
  }
  if ( cmd != 0xC0DE1337 )
    return -1LL;
  v6 = req.size;
  if ( req.size <= beta_storage.size )
  {
    memcpy(tempstorage, beta_storage.data, req.size);
    if ( req.transfer_data )
    {
      if ( v6 <= 0x30uLL )
        copy_to_user(req.transfer_data, tempstorage, v6);
    }
    printk(&unk_2C1);
LABEL_10:
    mutex_unlock(&safety_lock);
    return 1LL;
```

### Store
Giving `0x1337C0DE` frees the global variable if there is already a chunk. Malloc a chunk of size provided by user and then copies `size` bytes into it.  
The maximum size allowed is 0x7FFFFFFF.  

### Read
Giving `0x1337C0DE` copies `size` bytes onto stack (size 48) where `size` is provided by user. Next it copies this data from stack to user. This copy only  
happens if the size provided by user is less that 0x30.


From this, it is pretty obvious we have a huge stack overflow. Since the stack buffer size is only 48 and we can give a huge input, we can do a classic kernel rop  
to call `commit_cred(prepare_kernel_creds(0))`.

Next thing we need to utilize the overflow is a leak. For this, we can use the fact that return value of `copy_from_user` is not checked for failure.  
So the idea is to create a struct on heap which will contain pointers. Then free that struct and call `Store` with the same size. In this we will give an  
invalid memory so that copy_from_user fails. With this, we have an uninitilaized memory and we can leak kernel pointer.

After this, we can just do a normal kernel rop to call `commit_creds(prepare_kernel_cred(0))`. In this challenge, we had to
reduce the size of the binary. To do this, we used `diet libc` which ended up creating a binary with size 20k.

### Conclusion

Even though the challenge is an easy one, this is the first kernel challenge me and [Cyb0rG](https://twitter.com/_Cyb0rG) solved during a ctf. You can
find the exploit [here](https://github.com/souragc/CTFs/blob/master/htb20finals/driver/exp.c)

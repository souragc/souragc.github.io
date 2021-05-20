---
layout: post
title: FPAnalyze
date: 2021-05-20 19:06:17
summary: Tool made to find pointers.
tags:
 - project
 - ctf
---

This is a tool created by myself and [Cyb0rG](https://pwnverse.github.io/).

### Introduction

If you are a ctf player, you will know that sometimes to get the shell, we have to overwrite some pointer
which will be used during the execution of the program. This is usually `malloc_hook` or some other hooks.
But it is not guaranteed to work. This is the place where our tool can be used.

### Working

When the program starts, we are hijacking the `_init` function. First we find the required binary and libc
addresses from `/proc/self/maps`. Then we go through the writable memory of both the program and libc
and find any pointers which are in the executable range. If we find something, we overwrite that with
a number which will be index into the array where these pointers are stored.
After this, we register a segfault handler. When the program executes and if it tries to execute some
pointers, it will segfault since the original value was overwritten. From the segfault handler,
we get the `RIP` value which will be our index. We then replace the memory with it's original value and 
also prints out this value. Next time if the program tries to execute the same pointer, there won't be
any segfault. This ensures that a pointer will be found only once.

### Usage

There is a `run.sh` provided which exports the shared object files and executes the binary. Additional
information on how to use can be found on github.

### Sample

This is how it looks while running with a challenge from one of the ctfs.  
![sample](/imgs/fpanalyze/sample.png)

### Code

You can find the source code and instructions on how to use [here](https://github.com/teambi0s/FPAnalyze)

---
layout: post
title:  "How to Add New System Calls to Minix 3.1.8"
date:   2012-02-02
categories: OS
---


There are some tutorials on the Internet. However, they are not up-to-date. Currently Minix has the latest version 3.1.8. It has some changes, say a virtual file system. So a  newer tutorial is necessary. This article also includes a part for adding system calls to servers that are not visible to users. For example, users cannot call the functions in MFS directly because there is a middleware layer, VFS, between them. In order to call some function in MFS, one need to add a system call to MFS and then call it from VFS. The steps of adding system calls to MFS is slightly different.

I had an OSDI class last semester, which required us to do some Minix kernel modification. I learned a lot from it.

Here are the steps to add a new system call to Minix 3.1.8.

Suppose we need to add a system call, `setmgroup()`, to Process Manager(PM).

1. Find an unused entry in `/usr/src/include/minix/callnr.h`, add a line. For example

    ```
#define SETMGROUP 70
    ```
2. In `/usr/src/servers/pm/table.c`, find the corresponding entry (with same number) in the table and change its name to `setmgroup`
3. Add

    ```
_PROTOTYPE( int do_setmgroup, (void) );
    ```
to `/usr/src/servers/pm/proto.h`.

4. Add the function body of `do_setmgroup()` to `/usr/src/servers/pm/misc.c`.
5. Go to `/usr/src/servers/pm/`, and 

    ```
$make
    ``` 
    to check if it can pass the compilation.

6. create file `mcastlib.h` in `/usr/include/`
add the `setmgroup(int, int)` to this file, which includes `_syscall()`
7. Do

    ```
    cd /usr/src/tools
    make hdboot
    reboot
    ```
8. Test it by using the system call.

The `mcastlib.h` is like this:

```
#include <lib.h>
#include <unistd.h>
#define setmgroup _setmgroup
```

```
PUBLIC int setmgroup ( int _groupid )
{
    message m;
    m.m1_i1 = _groupid;
    return( _syscall( PM_PROC_NR, SETMGROUP, &m) );
}

```

<h2>Add a System Call to MFS</h2>
Suppose we need to add a system call named `listblocknum()` to MFS.
1. add an entry in `/usr/src/include/minix/vfsif.h`

    add this line:

    ```
#define REQ_LISTBLOCKNUM (VFS_BASE + 33)
    ```

    Add 1 to the number of request

    ```
    #define NREQS 34
    ```

2. find the entry in the table and change its name to `fs_listblocknum` in `/usr/src/servers/mfs/table.c`
3. find the entry in the table and change its name to `no_sys` in `/usr/src/servers/hgfs/table.c` and other file systems like ext2, etc.
    If you already changed NREQS and still got the error “array size is negative”. Try to “$make includes” in /usr/src. Then try to make in hgfs again.
4. add `_PROTOTYPE( int fs_listblocknum, (void) );` to `/usr/src/servers/mfs/proto.h`
5. add the function body of `fs_listblocknum()` to `/usr/src/servers/mfs/misc.c`
6. Do

    ```
    $cd /usr/src/servers/mfs/
    $make
    ```
7. go to `/usr/src/tools`,

    ```
    $make hdboot
    ```

After doing this, VFS can send message to MFS to call a function, just like USER can send a message to VFS to call a function. Refer to request.c in VFS to see how to call the functions in MFS.



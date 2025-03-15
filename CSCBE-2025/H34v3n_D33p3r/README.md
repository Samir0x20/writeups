## Challenge Description

Exhausted after two continuous days of restoring systems after the incident,
you come across an unknown file hidden deep within the root server.

It must contain the key to decrypt the mess created by that damned black hat,
you’re sure of it...

## Solution

We get 2 files. One of them content some tips and the other one is encoded in base64.

I used the command `cat unknown | base64 --decode > output`. We get a tar file then decompresse it.

After that, we get 2 files. One image and one linux executable. If we try to execute the ELF 64-bit LSB pie executable. We get this error:

```
┌──(kali㉿kali)-[~/…/rev/H34v3n_D33p3r/challenge_files_h34v3n_d33p3r (2)/output_FILES]
└─$ ./wzky                 
./wzky: error while loading shared libraries: libl98745.so: cannot open shared object file: No such file or directory
```

We are missing a strange shared object and the only place we can find something is from that image.

So I used binwalk to check if anything is hidden in that image.

```
┌──(kali㉿kali)-[~/…/rev/H34v3n_D33p3r/challenge_files_h34v3n_d33p3r (2)/output_FILES]
└─$ binwalk 65ab3g89k 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, EXIF standard
12            0xC             TIFF image data, big-endian, offset of first image directory: 8
256854        0x3EB56         ELF, 64-bit LSB shared object, AMD x86-64, version 1 (SYSV)
```

We can see that a shared object (.so) is hidden. I used this commandto extract this shared object: `dd if=65ab3g89k of=libl98745.so bs=1 skip=256854`

After reversing engineering both files, libl98745.so and wzky, using ghidra. We have 2 possibilities to get the flag. The first one is to satisfy all conditions from **wzky** or call the function decrypting the flag from **libl98745.so**.

The easiest one is to load libl98745.so and get the flag. For that I used this C code:

```c
#include <stdio.h>
#include <dlfcn.h>

int main() {
    // Load the shared library
    void *handle = dlopen("./libl98745.so", RTLD_LAZY);
    if (!handle) {
        fprintf(stderr, "Error opening library: %s\n", dlerror());
        return 1;
    }

    // Get the function pointer for the mangled name
    void (*asio_nsense)(void) = (void (*)(void))dlsym(handle, "_Z11asio_nsensev");

    // Check for errors
    char *error = dlerror();
    if (error != NULL) {
        fprintf(stderr, "Error finding symbol: %s\n", error);
        dlclose(handle);
        return 1;
    }

    // Call the function
    printf("Calling asio_nsense...\n");
    asio_nsense();

    // Close the library
    dlclose(handle);
    return 0;
}
```

**_Z11asio_nsensev** is the function name in this shared object.
```
┌──(kali㉿kali)-[~/…/rev/H34v3n_D33p3r/challenge_files_h34v3n_d33p3r (2)/output_FILES]
└─$ gcc -o flag flag.c -ldl
                                                                                                                                       
┌──(kali㉿kali)-[~/…/rev/H34v3n_D33p3r/challenge_files_h34v3n_d33p3r (2)/output_FILES]
└─$ ./flag     
Calling asio_nsense...
Your deserve this flag: CSC{G0t_4_Th1nG_L1k3_m0v_r4x_0x20_1n_mY_m1nD}

```

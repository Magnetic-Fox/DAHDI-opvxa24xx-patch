# opvxa24xx driver, DAHDI and DKMS configuration file patches

## Introduction

This repository contains my patches for `opvxa24xx` driver from OpenVox
and simple patches for DAHDI Kbuild and DKMS files (to add `opvxa24xx`
to the build list).

## Why was this created?

In fact, I didn't suppose I would have to create such patches, but when
I got interested in OpenVox A1610 and A2410 cards, I've found that
drivers for them are not included in the official Ubuntu/Debian package,
which simply meant those cards won't work after inserting them to my
server.

Of course, compiling the whole DAHDI package from openvoxtech.com would
be an overkill for me as I simply want to use official Debian packages
where possible (my server is running Debian), so I decided to try to
simply add `opvxa24xx` sources to the DAHDI sources and then look for
what's possible and what's not.

This wasn't that easy and the work had to be done, but fortunately
(after so many hard tries) it turned out to be much easier than it
looked at first.

## VERY IMPORTANT NOTE

...before going any further

> [!CAUTION]
> I've tested the `opvxa24xx` code **partially** only, because I don't
> have (yet) A1610E nor A2410E cards. They are quite expensive and
> I simply need time to be able to afford them.  
> For now, the driver compiles, DKMS can sign it and it can be loaded
> using `modprobe` and its information can be gathered using `modinfo`.  
> At this point I haven't got any kernel panic nor other strange
> behavior at the server, but I simply can't guarantee (yet) that
> everything would work as it should.  
> **YOU'RE USING THOSE SOLUTIONS AT YOUR OWN RISK!**

## Prerequisites

1. Sources of DAHDI from official Ubuntu/Debian package
   (`dahdi-2.11.1.0.20170917` usually seen in `/usr/src` after
   installing proper packages - I suggest installing those: `dahdi`,
   `dahdi-dkms`, `dahdi-firmware-nonfree`, `dahdi-linux`,
   `dahdi-source`)
   
2. Sources of DAHDI from openvoxtech.com
   (for `opvxa24xx` driver source - You can find it on the A1610/A2410
   description sites)
   
## Installation instructions

1.  As I've stated before, I suggest installing those packages (of
    course if You're using Ubuntu/Debian: `dahdi`, `dahdi-dkms`,
    `dahdi-firmware-nonfree`, `dahdi-linux`, `dahdi-source`), because
    they'll contain everything needed (especially `dahdi-dkms` and
    `dahdi-source`) - all the configuration files and `dkms.conf` and
    will be placed in the right place (`/usr/src`)
   
2.  While installing those packages, the first build will be made
    immediately by DKMS for Your kernel (yet without `opvxa24xx`)
   
3.  Now it's time for patching - unpack the `opvxa24xx` folder from the
    full DAHDI sources archive downloaded from openvoxtech.com and copy
    it to the `drivers/dahdi` folder (assuming that
    `dahdi-2.11.1.0.20170917` resides in the `/usr/src` directory, You
    should copy `opvxa24xx` folder to such path:
    `/usr/src/dahdi-2.11.1.0.20170917/drivers/dahdi`)
   
4.  Patch `Kbuild` file in the `opvxa24xx` directory - simply replace
    the file with that from mine repository (it contains patches for
    changed instructions, names and forgotten defines and `EXTRA_FLAGS`
    mapped to `ccflags-y`)

5.  (optional) If You use 32-bit OS You have to patch the `callerid.c`
    file using `opvxa24xx_32bit_ktime.patch` file to make linking of
    the driver for 32-bit processor possible (this patch changes the
    time division instruction from direct `/` to `div_s64` kernel
    procedure, which allows for 64-bit integer division in the 32-bit
    environment) - just do:
    
    `sudo patch callerid.c <opvxa24xx_32bit_ktime.patch`
   
6.  Inform the official DAHDI package of the new driver - patch `Kbuild`
    file in the `drivers/dahdi` folder - again, just replace the
    original file with that provided by me in this repository (it just
    adds `opvxa24xx` driver to the list of built drivers)
   
7.  For full automation, patch `dkms.conf` file to achieve automatic
    drivers recompilation (with `opvxa24xx` too) if Your kernel gets
    changed (during updates, etc.) - simply replace `dkms.conf` file
    in the main DAHDI source directory with the version provided by me
    in this repository (which, again, just adds `opvxa24xx` driver to
    the list)

8.  (optional, but recommended) Make full copy of the
    `dahdi-2.11.1.0.20170917` directory to Your home folder (or anything
    else) and try to compile everything using `make all` to test if
    everything is going right
   
9.  Remove, add, build and install the package "the official way", using
    DKMS - do as follows:
   
    a) `sudo dkms remove dahdi/2.11.1.0.20170917 --all`  
    b) `sudo dkms add dahdi/2.11.1.0.20170917`  
    c) `sudo dkms build dahdi/2.11.1.0.20170917`
   
    You may encounter errors at this point (DKMS might fail even while
    adding DAHDI again). In such case You'll need to manually remove
    compiled libraries:
   
    d) `sudo rm -rf /var/lib/dkms/dahdi/2.11.1.0.20170917/your_kernel`
   
    Now build would be possible, so repeat that step:
   
    e) `sudo dkms build dahdi/2.11.1.0.20170917`
   
    And now force the installation:
   
    f) `sudo dkms install dahdi/2.11.1.0.20170917 --force`

10. At this point, You should have official DAHDI compiled and installed
    and `opvxa24xx` driver available too (and prepared to recompile on
    any kernel changes) - You can test it:
    
    a) `sudo modprobe dahdi`  
    b) `sudo modprobe opvxa24xx`  
    c) `sudo modinfo opvxa24xx`

## Disclaimer

I've prepared this repository the way it should explain everything.
First commits are original files taken from Ubuntu/Debian packages and
from openvoxtech.com's full DAHDI source code archive and the later
ones are just mine updates, so You can see what changes were actually
made.

I hope this repository does not violate any licenses or laws - my
intention, while creating and sharing it, was just to help other people
which, just like me before, got stuck while deciding to buy
A1610/A2410 cards and got to know that, from various reasons, the
drivers for them are not available in the official packages.

Of course, as it was stated before, **I haven't tested this driver
properly yet**, but all the changes made are very simple and for now
it looks like everything should work correctly. I'll update this
README file when I got my A1610E card and would then know if it works
correctly or not. For now, I unfortunately need time to achieve this.

However, **You're using codes provided here at Your own risk!**
I don't take responsibility for anything.

## Copyrights and thanks

First of all, I'd love to thank the whole Debian/Ubuntu team for making
such a good piece of work making DAHDI codes compiling and working as
they should - I do really appreciate it.

I'd love to thank OpenVox too for creating such a great
telecommunication cards - I've always dreamed of creating my own PBX and
their cards just makes it possible and are still possible to buy as new.

Now, as far as I know:

* All the work done by Debian/Ubuntu teams belongs to those people, so
  any copyrights also belongs to them

* All the work done by people creating DAHDI belongs to them, so, like
  before, all copyrights also belongs to them

* OpenVox, their codes and hardware are work done by them, so copyrights
  belongs to them
  
## Patches author

I'm the author of all patches presented within this repository and I'm
releasing it here for free for anyone (just don't remove my nickname
from code comments and it'll be fine for me).

Hope this repository will help somebody.

Bartłomiej "Magnetic-Fox" Węgrzyn,  
October 20, 2026

![Heads booting on an x230](https://user-images.githubusercontent.com/827570/156627927-7239a936-e7b1-4ffb-9329-1c422dc70266.jpeg)

Heads: the other side of TAILS
===

Heads is a configuration for laptops and servers that tries to bring
more security to commodity hardware.  Among its goals are:

* Use free software on the boot path
* Move the root of trust into hardware (or at least the ROM bootblock)
* Measure and attest to the state of the firmware
* Measure and verify all filesystems

![Flashing Heads into the boot ROM](https://farm1.staticflickr.com/553/30969183324_c31d8f2dee_z_d.jpg)

NOTE: It is a work in progress and not yet ready for non-technical users.
If you're interested in contributing, please get in touch.
Installation requires disassembly of your laptop or server,
external SPI flash programmers, possible risk of destruction and
significant frustration.

More information is available in [the 33C3 presentation of building "Slightly more secure systems"](https://trmm.net/Heads_33c3).

Documentation
===
Please refer to [Heads-wiki](https://osresearch.net) for your Heads' documentation needs.


Building heads
===
`make BOARD=board_name` where board_name is the name of the board directory under `./boards` directory.

In order to build reproducible firmware images, Heads builds a specific
version of gcc and uses it to compile the Linux kernel and various tools
that go into the initrd.  Unfortunately this means the first step is a
little slow since it will clone the `musl-cross-make` tree and build gcc...

Once that is done, the top level `Makefile` will handle most of the
remaining details -- it downloads the various packages, verifies the
hashes, applies Heads specific patches, configures and builds them
with the cross compiler, and then copies the necessary parts into
the `initrd` directory.

There are still dependencies on the build system's coreutils in
`/bin` and `/usr/bin/`, but any problems should be detectable if you
end up with a different hash than the official builds.

The various components that are downloaded are in the `./modules`
directory and include:

* [musl-libc](https://www.musl-libc.org/)
* [busybox](https://busybox.net/)
* [kexec](https://wiki.archlinux.org/index.php/kexec)
* [mbedtls](https://tls.mbed.org/)
* [tpmtotp](https://trmm.net/Tpmtotp)
* [coreboot](https://www.coreboot.org/)
* [cryptsetup](https://gitlab.com/cryptsetup/cryptsetup)
* [lvm2](https://sourceware.org/lvm2/)
* [gnupg](https://www.gnupg.org/)
* [Linux kernel](https://kernel.org)

We also recommend installing [Qubes OS](https://www.qubes-os.org/),
although there Heads can `kexec` into any Linux or
[multiboot](https://www.gnu.org/software/grub/manual/multiboot/multiboot.html)
kernel.

Notes:
---

* Building coreboot's cross compilers can take a while.  Luckily this is only done once.
* Builds are finally reproducible! The [reproduciblebuilds tag](https://github.com/osresearch/heads/issues?q=is%3Aopen+is%3Aissue+milestone%3Areproduciblebuilds) tracks any regressions.
* Currently only tested in QEMU, the Thinkpad x230, Librem series and the Chell Chromebook.
** Xen does not work in QEMU.  Signing, HOTP, and TOTP do work; see below.
* Building for the Lenovo X220 requires binary blobs to be placed in the blobs/x220/ folder.
See the readme.md file in that folder
* Building for the Librem 13 v2/v3 or Librem 15 v3/v4 requires binary blobs to be placed in
the blobs/librem_skl folder. See the readme.md file in that folder

QEMU:
---

OS booting can be tested in QEMU using a software TPM.  HOTP can be tested by forwarding a USB token from the host to the guest.

For more information and setup instructions, refer to the [qemu-coreboot-fbwhiptail-tpm1-hotp documentation](boards/qemu-coreboot-fbwhiptail-tpm1-hotp/qemu-coreboot-fbwhiptail-tpm1-hotp.md).

coreboot console messages
---
The coreboot console messages are stored in the CBMEM region
and can be read by the Linux payload with the `cbmem --console | less`
command.  There is lots of interesting data about the state of the
system.

# Simple KVM firmware

**This project is an experiment and should not be used production workloads.**

This repository contains a simple KVM firmware that is designed to be launched
from anything that supports loading ELF binaries and running them with the Linux
kernel loading standard.

The ultimate goal is to be able to use this "firmware" to be able to load a
bootloader from within a disk image.

Currently it will directly load a kernel from a disk image that follows the
[Boot Loader Specification](https://systemd.io/BOOT_LOADER_SPECIFICATION)

Although this project has been developed using
[Firecracker](https://github.com/firecracker-microvm) as it does not currently
support resetting the virtio block device it is not possible to boot all the
way into the OS.

## Building

To compile:

cargo xbuild --release --target target.json

The result will be in:

target/target/release/hypervisor-fw

Debug builds do not currently function.

## Features

* virtio (MMIO & PCI) block support
* GPT parsing (to find EFI system partition)
* FAT12/16/32 directory traversal and file reading
* bzImage loader
* "Boot Loader Specification" parser

## Running

Works with Firecracker as a drop in replacement for the Linux kernel. It does
not work with crosvm as crosvm has a hardcoded kernel function start address.

### Firecracker

As per [quick start](https://github.com/firecracker-microvm/firecracker/blob/master/docs/getting-started.md)

Replacing the kernel and rootfs to point at the firmware and the full disk
image instead.

```
curl --unix-socket /tmp/firecracker.socket -i \
    -X PUT 'http://localhost/boot-source'   \
    -H 'Accept: application/json'           \
    -H 'Content-Type: application/json'     \
    -d '{
        "kernel_image_path": "target/target/release/hypervisor-fw",
        "boot_args": ""
    }'

curl --unix-socket /tmp/firecracker.socket -i \
    -X PUT 'http://localhost/drives/rootfs' \
    -H 'Accept: application/json'           \
    -H 'Content-Type: application/json'     \
    -d '{
        "drive_id": "rootfs",
        "path_on_host": "clear-28660-kvm.img",
        "is_root_device": true,
        "is_read_only": false
    }'

curl --unix-socket /tmp/firecracker.socket -i \
    -X PUT 'http://localhost/actions'       \
    -H  'Accept: application/json'          \
    -H  'Content-Type: application/json'    \
    -d '{
        "action_type": "InstanceStart"
     }'

```

**Currently Firecracker's virtio block device does not support resetting the
device and as such it is not possible for the booted Linux kernel to take over
the device from the firmware.**

## Testing

"cargo test" needs disk images from make-test-disks.sh

And clear-28660-kvm.img:

https://download.clearlinux.org/releases/28660/clear/clear-28660-kvm.img.xz

sha1sum: 5fc086643dea4b20c59a795a262e0d2400fab15f

## TODO

* PE32 loader
* EFI runtime services required for booting

## Security

**Reporting a Potential Security Vulnerability**: If you have discovered
potential security vulnerability in this project, please send an e-mail to
secure@intel.com. For issues related to Intel Products, please visit
https://security-center.intel.com.

It is important to include the following details:
  - The projects and versions affected
  - Detailed description of the vulnerability
  - Information on known exploits

Vulnerability information is extremely sensitive. Please encrypt all security
vulnerability reports using our *PGP key*

A member of the Intel Product Security Team will review your e-mail and
contact you to to collaborate on resolving the issue. For more information on
how Intel works to resolve security issues, see: *Vulnerability Handling
Guidelines*

PGP Key: https://www.intel.com/content/www/us/en/security-center/pgp-public-key.html

Vulnerability Handling Guidelines: https://www.intel.com/content/www/us/en/security-center/vulnerability-handling-guidelines.html


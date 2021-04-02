# Linux rootfs for RISC-V

This is a tiny RISC-V Linux root filesystem image that targets
the `virt` machine in riscv-qemu.

The root image is intended to demonstrate virtio-net and virtio-block in
riscv-qemu and features a dropbear ssh server which allows out-of-the-box
ssh access to a RISC-V virtual machine.

## Dependencies

- [riscv-gnu-toolchain](https://github.com/riscv/riscv-gnu-toolchain) (RISC-V Linux toolchain, recommended with Linux-elf/glibc)
- [busybox](https://busybox.net/) (downloaded automatically)
- [dropbear](https://matt.ucc.asn.au/dropbear/dropbear.html) (downloaded automatically)
- sudo, curl, openssl and rsync

## Features

- `ntpd` for time configuration
- `klog` for kernel logging
- `syslog` for system logging
- `dropbear` for ssh access
- `busybox` with almost everything enabled

## Configuration

- `conf` contains the linux image build configuration
  - `conf/linux.config` contains the linux-kernel configuration
  - `conf/busybox.config` contains the busybox configuration
  - `conf/riscv64.config` contains the image build configuration
    - `ROOT_PASSWORD=oscomp`
    - `IMAGE_FILE=riscv64-rootfs.bin`
- `etc` contains the linux guest system configuration
  - `etc/network/interfaces` (guest 192.168.100.2, router 192.168.100.1)
  - `etc/resolv.conf` (nameserver 8.8.8.8, nameserver 8.8.4.4)
  - `etc/ntp.conf` (server 0.pool.ntp.org, server 1.pool.ntp.org)
  - `etc/passwd`, `etc/shadow`, `etc/group` and `etc/hosts`

The default config assumes bridged networking with `192.168.100.1`
on the host and `192.168.100.2` in the guest.

## Build

The build process downloads busybox and dropbear, compiles them and prepares
a root filesystem image to the file `riscv64-rootfs.bin`.

```
export PATH=/path/to/your/riscv/toolchain:$PATH
make
```
which runs executes this script: `scripts/build.sh`

## Running

Requires the riscv-qemu `virt` board with virtio-block and virtio-net devices.

The following command starts riscv64 Linux:

```
make run
```

which runs executes this command:

```
sudo qemu-system-riscv64 -nographic -machine virt \
  -kernel bbl -append "root=/dev/vda ro console=ttyS0" \
  -drive file=riscv64-rootfs.bin,format=raw,id=hd0 \
  -device virtio-blk-device,drive=hd0 \
  -netdev type=tap,script=scripts/ifup.sh,downscript=scripts/ifdown.sh,id=net0 \
  -device virtio-net-device,netdev=net0
```

After booting the virtual machine you should be able to ssh into it.

```
$ ssh root@192.168.100.2
```
Through `scp`, Host and Qemu guest can transfer files.
---
OR Just use my [prebuilt](#)
---
* 分区/dev/vda2是fat32分区，挂载后可模拟比赛时的sdcard fat32

### k210运行linux

对于k210板子运行Linux，由于内存所限，暂时使用的是riscv64 uclibc-nommu版本

* 先构建riscv64 nommu uClibc toolchain的工具链<br>
可参照该[buildroot](https://github.com/damien-lemoal/riscv64-nommu-buildroot)

* 基于uClibc工具链，构建root文件系统<br>
基于busybox来构建，最后生成cpio包：`cd rootfs && find . | cpio -H newc -o > ../k210.cpio`<br>
可以使用已经编译好的[k210.cpio](https://cloud.tsinghua.edu.cn/d/216d048d297444ee96e4/files/?p=%2Fk210.cpio)

* 然后构建Linux Kernel<br>
目前Linux主分支已经支持k210开发版，可选择nommu_k210_defconfig
此时把文件系统包含进内核，如内核配置：`CONFIG_INITRAMFS_SOURCE="k210.cpio"`
可以使用已经编译好的[Kernel](https://cloud.tsinghua.edu.cn/d/216d048d297444ee96e4/files/?p=%2Fk210.bin)

* 烧写进k210板子
`kflash.py -t -b 3000000 -p /dev/ttyUSB0 arch/riscv/boot/loader.bin`

### 测试用例
[测试用例](https://github.com/oscomp/syscalls_testcases_ltp.git)同样是基于uClibc工具链来构建的<br>
由于内存所限, no MMU, 一些较基本的syscall会无法使用，如fork()、clone()等<br>
这导致测试用例有很多是无法运行的，运行时会出现crash<br><br>
在测试多个测试用例时有个小技巧，可以不用每次重复烧写k210板子：
- Minicom的串口传文件的功能，Ctrl-a，s可以选择从Host传入k210板子内的文件
- 编译lrzsz进rootfs的`usr/bin/rz`, `usr/bin/sz`
- 串口传文件一次不要传太多，容易卡住；


不过有一部分的测试用例是[可以运行的](https://cloud.tsinghua.edu.cn/d/216d048d297444ee96e4/files/?p=%2Fk210-uclibc-testcases.tgz)
已经试验的列表如下(都是一个一个打进文件系统烧写啊%>_<%)

```
umask01   
unlink07
uname01
setgid01
sethostname01
timers01
truncate01
symlink03
sigsuspend01
signal03
setuid01
settimeofday01
rmdir01
reboot01
read01
pselect01
prctl01
poll01
pipe01
pidfd_open01
open01
ioctl01
lstat01
linkat01
link02
gettimeofday02
gethostname01
getrlimit01
getpgid01
getitimer01
getgroups01
getcwd01
getcpu01
ftruncate01
fstatat01
fstat02
fpathconf01
fchown02
fchmod01
fchdir01
faccessat01
chroot03
alarm02
clock_settime01
add_key02
acct01
access02
accept02
```


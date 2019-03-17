# XDP Experiments

## xdpsock

How to build the sample xdpsock application (see https://github.com/torvalds/linux/blob/master/samples/bpf/xdpsock_user.c) on archlinux.

NOTE: instructions borrowed from https://github.com/pavel-odintsov/fastnetmon/wiki/af_xdp-tests-for-Linux-4.19

1. Download the appropriate kernel version:
  - See http://cdn.kernel.org/pub/linux/kernel/v5.x

2. Prepare kernel (use config.h from official kernel)
```
make mrproper
cp /usr/lib/modules/$(uname -r)/build/.config ./
cp /usr/lib/modules/$(uname -r)/build/Module.symvers ./
make oldconfig
```

3. Compile prerequisites
```
cd linux/tools/lib/bpf
make
export CFLAGS="-I../../include/uapi -I../../include -I../../tools/lib -I../../tools/testing/selftests/bpf"
```

4. Compile userspace app
```
cd linux/samples/bpf
LDFLAGS="-L../../tools/lib/bpf -lbpf -lelf -lpthread" make xdpsock_user
```

5. Compile kernelspace app
```
cd linux/samples/bpf
clang $(echo $CFLAGS) -fno-stack-protector -c xdpsock_kern.c -D__KERNEL__ -D__BPF_TRACING__ -Wno-unused-value -Wno-pointer-sign  -Wno-compare-distinct-pointer-types -Wno-gnu-variable-sized-type-not-at-end -Wno-address-of-packed-member -Wno-tautological-compare -Wno-unknown-warning-option -O2 -emit-llvm -c -o -| llc -march=bpf -filetype=obj  -o xdpsock_user_kern.o
```

6. Run
```
sudo LD_LIBRARY_PATH=../../tools/lib/bpf ./xdpsock_user -i wlan0
```

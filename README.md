```
$ cd sdk-qemu-aarch64/

$ docker build --squash -t sdk-qemu-aarch64 .
```

Setup `qemu-user` requirements on host

```
$ cd ViryaOS/sdk-qemu-aarch64/

$ sudo ./setup_qemu-user_host.sh

$ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
```

Go to the directory containing `ViryaOS` tree.

```
$ docker run --rm --privileged=true -ti -v $(pwd):/home/builder/src -v /tmp:/tmp \
       sdk-qemu-aarch64 /bin/su -l -s /bin/sh builder
```

NOTE: *DO NOT* run

```
$ sudo apk update
$ sudo apk add <package>
```

Inside the container, as it will downgrade `qemu-aarch64` and
`qemu-system-aarch64`. If you need to install  packages inside the container,
do it via `Dockerfile`.

To create a aarch64 chroot

```
$ abuild-apk --arch aarch64 --root /home/builder/rootfs-aarch64/ add --initdb

$ sudo mkdir -p /home/builder/rootfs-aarch64/etc/apk/keys

$ sudo cp /usr/share/apk/keys/aarch64/alpine-devel*.rsa.pub \
    /home/builder/rootfs-aarch64/etc/apk/keys/

$ sudo mkdir -p /home/builder/rootfs-aarch64/usr/bin

$ sudo cp /usr/bin/qemu-aarch64 /home/builder/rootfs-aarch64/usr/bin/

(You will get {pre,post}-install script errors if qemu-aarch64 is not copied
over!)

$ sudo cp /etc/resolv.conf /home/builder/rootfs-aarch64/etc/resolv.conf

$ sudo mount --bind /proc /home/builder/rootfs-aarch64/proc

$ abuild-apk update --arch aarch64 --root /home/builder/rootfs-aarch64/ \
    --repository http://dl-cdn.alpinelinux.org/alpine/v3.7/main

$ abuild-apk --arch aarch64 --root /home/builder/rootfs-aarch64/ \
    --repository http://dl-cdn.alpinelinux.org/alpine/v3.7/main  \
    update

$ abuild-apk --arch aarch64 --root /home/builder/rootfs-aarch64/ \
   --repository http://dl-cdn.alpinelinux.org/alpine/v3.7/main   \
   add alpine-baselayout busybox apk-tools

(Specify any additional package you might need after `busybox`. There should be
no line break after add)

$ sudo chroot /home/builder/rootfs-aarch64 /bin/sh
```

Once you are done with the chroot

```
$ sudo umount /home/builder/rootfs-aarch64/proc

$ sudo rm -f /home/builder/rootfs-aarch64/usr/bin/qemu-aarch64

$ sudo rm -rf /home/builder/rootfs-aarch64
```

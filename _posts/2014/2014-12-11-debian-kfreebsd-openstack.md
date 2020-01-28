---
title: Debian-kFreeBSD inside OpenStack
---

I don't know about other OpenStack installations, to be honest, but at least the one we have at work uses virtio, for which Debian-kFreeBSD (Wheezy) does not include drivers.

So if you try to create an image using kvm as this

```bash
kvm -smp 1 -m 512 -cdrom debian-7.7.0-kfreebsd-amd64-netinst.iso -drive if=virtio,file=debian-kfreebsd-x86_64.img -net nic,model=virtio -net user
```
The system will not manage to find the hard drive, and will fail miserably when trying to partition.

You could, of course, do the installation without virtio, but then the resulting image will not boot in OpenStack.

So here is what I did. Please, mind I am not a FreeBSD expert, neither an OpenStack one, so probably this wasn't the best way!

## 1. Install without virtio
```bash
kvm -smp 1 -m 512 -cdrom debian-7.7.0-kfreebsd-amd64-netinst.iso -drive file=debian-kfreebsd-x86_64.img -net nic -net user
```
## 2. (Re)Boot the installtion without virtio
Same thing, but you can skip the cdrom :-)

## 3. Download FreeBSD virtio drivers
I have seen variations of [these instructions](https://wiki.znode.net/pub/Debian/KFreeBSD) (invalid certificate) in a couple of other places, but they do not work any more. Mainly because the link is broken.

You can get them now from [the maintainer web page (~kuriyama)](http://people.freebsd.org/~kuriyama/virtio/), but mind to choose the version that matches your kernel! For Wheezy, at least the one I used, it has to be the 9.0 release. 9.1 would not even load.

```bash
wget http://people.freebsd.org/~kuriyama/virtio/9.0/virtio-kmod-9.0-0.250249.tbz
mkdir virtio
tar xvf virtio-kmod-9.0-0.250249.tbz  -C virtio
```

And copy them to the directory where the kernel modules are installed

```bash
cp virtio/boot/modules/*.ko /lib/modules/9.0-2-amd64/
```

You can double check if they are the right version, architecture... loading virtio.ko, for instance

```bash
$ kldload virtio
$ kldstat
1   13 0xffffffff80200000 d71000   kfreebsd-9.0-2-amd64.gz
2    4 0xffffffff81136000 439d     virtio.ko
...
```
If you get something similar to that output, you got the right modules.

## 4. Enable virtio modules at boot time
I just added after this line
```bash
load_kfreebsd_module acpi true
```

These lines
```bash
  # virtio modules
  for mod in virtio virtio_pci if_vtnet virtio_blk; do
    load_kfreebsd_module $mod false
  done
```
Then did `update-grub` and that's it. You can double check it did work having a look at `/boot/grub/grub.cfg`

## 5. Fix the mess
Now booting with virtio works

```bash
kvm -smp 1 -m 512 -cdrom debian-7.7.0-kfreebsd-amd64-netinst.iso -drive if=virtio,file=debian-kfreebsd-x86_64.img -net nic,model=virtio -net user
```

Only not really. If you do that the kernel will now see the hardware, but since changing the drivers changed the interface names it will ask you which root (`/`) device should be used.

Basically, if the root partition was `/dev/ada0s2`, now it will be `/dev/vtbd0s2`. Same applies to the others **and the netwotk interface**, which now will be called `vtnet0`.

I would recomment fixing this in your `/etc/fstab` and `/etc/network/interfaces` files before rebooting.

## 6. Install cloud-init
```bash
apt-get install cloud-init
```
And just keep going as usual ;-)

P.S At least the default installation of cloud-init seems to disable ssh access to the `root` user, and put your ssh key on the `debian` user account, but it doesn't give it sudo access (actually if I remember correctly sudo is not even installed by default), and I think it changes, or removes, the password, so before uploading to OpenStack, remember to create this user, configure it to your needs, and add it to the `sudo` group! Also, you may want to configure sudo to not to ask for a password when the user does belong to the `sudo` group, something like this:

```
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

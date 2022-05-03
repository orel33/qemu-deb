# Rebuild Qemu Package for Ubuntu

Recent Ubuntu 20/22 provides their *Qemu* package without VDE Support for
obscure reasons (*vde is debian-only since ubuntu/vde2 is in universe*). So to
overcome this issue, we just propose to rebuild this package for Ubuntu with VDE
support.

## Requirements

Before to start...

```bash
sudo apt install build-essential fakeroot devscripts
```

## Step by Step

First install the regular debian package for Qemu.

```bash
sudo apt install qemu-system-x86
```

### Get sources

Edit `/etc/apt/sources.list` if required, then get the package sources in the
current directory.

```bash
sudo apt-get update
sudo apt-get source qemu-system-x86
```

Get build dependencies.

```bash
sudo apt-get build-dep qemu-system-x86
```

### Enable VDE support for Qemu

Here, we just want to add the `--enable-vde` option to the *configure* command
line called by the `debian/rules` script.

In practice, the package dependencies are listed in the `debian/control` file,
with some comments to specify (system-specific) arguments to qemu's configure
script (as for instance `# --enable-xxx`).

In our case, we just need to remove he `:debian:` prefix from the template file
`debian/control-in` in the following lines:

```
# vde is debian-only since ubuntu/vde2 is in universe
:debian:# --enable-vde
:debian: libvdeplug-dev,
```

Then, one can update `debian/control` as follows:

```bash
sudo ./debian/rules debian/control
```

### Rebuild

Then, rebuild the package...

```bash
cd qemu-6.2+dfsg/
sudo debuild -us -uc
```

Be patient, it is really really long...

**Nota Bene** : The `-us -uc` options avoid the signature step in the build
process...

The build log are available in `qemu_6.2+dfsg-2ubuntu6_amd64.build`.

### Install

Check you have already installed the *official* qemu package
(`qemu-system-x86`), then update this with the new generated packages as follows:

```bash
sudo apt install qemu-system-x86
sudo dpkg -iO *.deb
```

**Nota Bene** : The `-O` option stands for `--selected-only`. Only process the
packages that are already installed.

Finally, if you want to prevent the package from being automatically installed,
upgraded or removed, you must set this package *hold*, as follows:

```bash
sudo apt-mark hold qemu-system-x86
```

### And it works

```bash
$ qemu-system-x86_64 -version
QEMU emulator version 6.2.0 (Debian 1:6.2+dfsg-2ubuntu6)
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers

$ qemu-system-x86_64 -netdev help
Available netdev backend types:
...
vde
...
```

## Documentation

* <https://wiki.debian.org/BuildingTutorial>
* <https://raphaelhertzog.com/2010/12/15/howto-to-rebuild-debian-packages/>
* <https://www.ducea.com/2008/03/06/howto-recompile-debian-packages/>

---
aurelien.esnard@u-bordeaux.fr

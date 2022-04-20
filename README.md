# Rebuild Qemu Debian Package

Recent Ubuntu 20/22 provides *Qemu* packages without VDE Support for obscure
reasons (vde is debian-only since ubuntu/vde2 is in universe). So to overcome
this issue, we just propose to rebuild this package.

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

Then, one can update `debian/control` file by launching the `./debian/rules`
script.

### Rebuild and Install

Then, rebuild the package...

```bash
cd qemu-6.2+dfsg/
sudo debuild -us -uc
```

Then, the build log are available in `qemu_6.2+dfsg-2ubuntu6_amd64.build`.


**Nota Bene** : The `-us -uc` options avoid the signature step in the build
process...

And install it...

```bash
sudo dpkg -i qemu_6.2+dfsg-2ubuntu6_amd64.deb
```

## Documentation

* <https://raphaelhertzog.com/2010/12/15/howto-to-rebuild-debian-packages/>
* <https://wiki.debian.org/BuildingTutorial>

---
aurelien.esnard@u-bordeaux.fr

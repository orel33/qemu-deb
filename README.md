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

### Do whatever changes you need

Here, we just want to add the `--enable-vde` option to the *configure* command
line in file the `debian/rules`. Visit the last section to see how to fix that.

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

## Fix VDE Support for Qemu

*In french only.*

Dans notre cas, il faut remarquer que le fichier `debian/rules` utilise un
script de la forme `./extract-config-opts amd64 control` pour extraire la liste
des options `--enable-xxx` à partir du fichier `debian/control` qui contrôle les
dépendances en fonction de l'architecture visée... En pratique, il faut recopier
les lignes suivantes du fichier `debian/control-in` dans le fichier
`debian/control` en prenant soin d'enlever le préfixe `:debian:`.

```
# vde is debian-only since ubuntu/vde2 is in universe
:debian:# --enable-vde
:debian: libvdeplug-dev,
```

Et donc, il faut rajouter dans le fichier `debian/control`, les lignes suivantes
:

```
# vde is debian-only since ubuntu/vde2 is in universe
# --enable-vde
 libvdeplug-dev,
```

## Documentation

* <https://raphaelhertzog.com/2010/12/15/howto-to-rebuild-debian-packages/>
* <https://wiki.debian.org/BuildingTutorial>

---
aurelien.esnard@u-bordeaux.fr

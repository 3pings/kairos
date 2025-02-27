---
layout: "../../layouts/docs/Layout.astro"
title: "Raspberry Pi"
index: 6
---

# Raspberry Pi support

Kairos supports Raspberry Pi model 3 and 4 with 64bit architecture.

If you are not familiar with the process, it is suggested to follow the [quickstart](/quickstart/installation) first to see how Kairos works.

## Prerequisites

- An SD card which size is at least 16 GB
- Etcher or `dd`
- A Linux host where to flash the device

## Download

Download the Kairos images from the [Releases](https://github.com/kairos-io/provider-kairos/releases) you are interested in. For example, for RPI and `k3sv1.21.14+k3s1`:

```bash
wget https://github.com/kairos-io/provider-kairos/releases/download/v1.0.0-rc2/kairos-opensuse-arm-rpi-v1.0.0-rc2-k3sv1.21.14+k3s1.img
```

## Flash the image

Plug the SD card to your system. To flash the image, you can either use Etcher or `dd`. Note it's compressed with "XZ", so we need to decompress it first:

```bash
xzcat kairos-opensuse-arm-rpi-v1.0.0-rc2-k3sv1.21.14+k3s1.img | sudo dd of=<device> oflag=sync status=progress bs=10MB
```

## Boot

Use the SD Card to boot. The default username/password is `kairos`/`kairos`.
To configure your access or disable password change the `/usr/local/cloud-config/01_defaults.yaml` accordingly.

## Configure your node

To configure the device beforehand, be sure to have the SD plugged in your host. We need to copy a configuration file into `cloud-config` in the `COS_PERSISTENT` partition:

```
$ PERSISTENT=$(blkid -L COS_PERSISTENT)
$ mkdir /tmp/persistent
$ sudo mount $PERSISTENT /tmp/persistent
$ sudo mkdir /tmp/persistent/cloud-config
$ sudo cp cloud-config.yaml /tmp/persistent/cloud-config
$ sudo umount /tmp/persistent
```

You can push additional `cloud config` files. For a full reference check out the [docs](/reference/configuration) and also [configuration after-installation](/advanced/after-install)

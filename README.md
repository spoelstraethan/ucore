# uCore

[![build-ucore](https://github.com/ublue-os/ucore/actions/workflows/build.yml/badge.svg)](https://github.com/ublue-os/ucore/actions/workflows/build.yml)

## What is this?

You should be familiar with [Fedora CoreOS](https://getfedora.org/coreos/), as this is an OCI image of CoreOS with "batteries included". More specifically, it's an opinionated, custom CoreOS image, built daily with some commonly used tools added in. The idea is to make a lightweight server image including most used services or the building blocks to host them.

WARNING: This image has **not** been heavily tested, though the underlying components have. Please take a look at the included modifications and help test if this project interests you.

## Features

`ucore` images:

- Start with a [Fedora CoreOS image](https://quay.io/repository/fedora/fedora-coreos?tab=tags)
- Remove stock packages:
  - toolbox
- Add the following:
  - [cockpit](https://cockpit-project.org)
  - [distrobox](https://github.com/89luca89/distrobox)
  - [duperemove](https://github.com/markfasheh/duperemove)
  - moby-engine(docker), docker-compose and podman-compose
  - [tailscale](https://tailscale.com) and [wireguard-tools](https://www.wireguard.com)
  - [ZFS](https://openzfs.github.io/openzfs-docs/Getting%20Started/Fedora/index.html)
- Enable staging of automatic system updates via rpm-ostreed
- Disable Zincati auto upgrade/reboot serivce
- Set a 60 second service stop timeout for reasonably fast shutdowns
- Enable password based SSH auth (required for locally running cockpit web interface)
- Suitable for use on bare metal or virtual machines to run containerized workloads

`ucore-hci` images:

- Start with `ucore` to give you everything above, PLUS:
- Add the following:
  - libvirt-daemon-kvm: KVM hypervisor management
  - virt-install: command-line utility for installing virtual machines
  - libvirt-client: `virsh` command-line utility for managing virtual machines
  - cockpit-machines: Cockpit GUI for managing virtual machines
- Suitable for use on bare metal to run as a hypervisor in addition to running containerized workloads

Note: per [cockpit instructions](https://cockpit-project.org/running.html#coreos) the cockpit-ws RPM is **not** installed, rather it is provided as a pre-defined systemd service which runs a podman container.

## Tips and Tricks

### Immutability and Podman

These images are immutable, you can't, and really shouldn't, install packages like in a mutable "normal" distribution.

CoreOS expects the user to run services using [podman](https://podman.io). `moby-engine`, the free Docker implementation, is installed for those who desire docker instead of podman.

### Default Services

To maintain this image's suitability as a minimal container host, most add-on services are not auto-enabled.

To activate any of the pre-installed `cockpit`, `docker`, `libvirtd`, `tailscaled` services, etc:

```bash
sudo systemctl enable --now SERVICENAME.service
```

### Docker/Moby and Podman

NOTE: CoreOS [cautions against](https://docs.fedoraproject.org/en-US/fedora-coreos/faq/#_can_i_run_containers_via_docker_and_podman_at_the_same_time) running podman and docker containers at the same time.  Thus, `docker.socket` is disabled by default to prevent accidental activate of docker daemon, given podman is the default.

### Distrobox

Users may use [distrobox](https://github.com/89luca89/distrobox) to run images of mutable distributions where applications can be installed with traditional package managers. This may be useful for installing interactive utilities such has `htop`, `nmap`, etc. As stated above, however, *services* should run as containers.

### CoreOS and ostree Docs

It's a good idea to become familar with the [Fedora CoreOS Documentation](https://docs.fedoraproject.org/en-US/fedora-coreos/) as well as the [CoreOS rpm-ostree docs](https://coreos.github.io/rpm-ostree/). Note especially, this image is only possible due to [ostree native containers](https://coreos.github.io/rpm-ostree/container/).

### ZFS

The ZFS kernel module and tools are pre-installed, but like other services, ZFS is not pre-configured to load on default.

Load it with the command `modprobe zfs` and use `zfs` and `zpool` commands as desired.

Per the [OpenZFS Fedora documentation](https://openzfs.github.io/openzfs-docs/Getting%20Started/Fedora/index.html):

> By default ZFS kernel modules are loaded upon detecting a pool. To always load the modules at boot:

```
echo zfs > /etc/modules-load.d/zfs.conf
```

## How to Install

### Prerequsites

This image is not currently avaialable for direct install. The user must follow the [CoreOS installation guide](https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/). There are varying methods of installation for bare metal, cloud providers, and virtualization platforms.

All CoreOS installation methods require the user to [produce an Ignition file](https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/). This Ignition file should, at mimimum, set a password and SSH key for the default user (default username is `core`).

### Install and Manually Rebase

You can rebase any Fedora CoreOS x86_64 installation to uCore.  Installing CoreOS itself can be done through [a number of provisioning methods](https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/).

To rebase an Fedora CoreOS machine to the latest uCore (stable):

1. Execute the desired `rpm-ostree rebase` command... (below)
1. Reboot, as instructed.
1. After rebooting, you should [pin the working deployment](https://docs.fedoraproject.org/en-US/fedora-silverblue/faq/#_how_can_i_upgrade_my_system_to_the_next_major_version_for_instance_rawhide_or_an_upcoming_fedora_release_branch_while_keeping_my_current_deployment) which allows you to rollback if required.

`ucore` stable stream

```bash
sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/ublue-os/ucore:stable
```

`ucore-hci` stable stream

```bash
sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/ublue-os/ucore-hci:stable
```

`ucore` testing stream

```bash
sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/ublue-os/ucore:testing
```

`ucore-hci` testing stream

```bash
sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/ublue-os/ucore-hci:testing
```

### Install with Auto-Rebase

Your path to a running uCore can be shortend by using [examples/ucore-autorebase.butane](examples/ucore-autorebase.butane) as the starting point for your CoreOS ignition file.

1. As usual, you'll need to [follow the docs to setup a password](https://coreos.github.io/butane/examples/#using-password-authentication). Substitute your password hash for `YOUR_GOOD_PASSWORD_HASH_HERE` in the `ucore-autorebase.butane` file, and add your ssh pub key while you are at it.
1. Generate an ignition file from your new `ucore-autorebase.butane` [using the butane utility](https://coreos.github.io/butane/getting-started/).
1. Now install CoreOS for [hypervisor, cloud provider or bare-metal](https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/). Your ignition file should work for any platform, auto-rebasing to the `ucore:stable`, rebooting and leaving your install ready to use.

## Verification

These images are signed with sisgstore's [cosign](https://docs.sigstore.dev/cosign/overview/). You can verify the signature by downloading the `cosign.pub` key from this repo and running the following command:

```bash
cosign verify --key cosign.pub ghcr.io/ublue-os/ucore
```

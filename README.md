<h1 align="center">libvirt for droidtop</h1>

<p align="center">
  <b>Virtual machines on Android, the normal libvirt way.</b><br>
  A root module that runs libvirt and QEMU, and a droidtop plugin that manages the machines.
</p>

<p align="center">
  <img alt="For droidtop" src="https://img.shields.io/badge/for-droidtop-2f7bf5?style=flat-square">
  <a href="LICENSE"><img alt="License: GPL-3.0" src="https://img.shields.io/badge/license-GPL--3.0-blue?style=flat-square"></a>
  <img alt="Status: design only" src="https://img.shields.io/badge/status-design%20only-d69e2e?style=flat-square">
</p>

<p align="center">
  <a href="#status">Status</a> ·
  <a href="#the-plan">The plan</a> ·
  <a href="#acceleration">Acceleration</a> ·
  <a href="#licence">Licence</a>
</p>

> **Design only.** There is no module and no plugin to install yet. This repository holds the design in [docs/DESIGN.md](docs/DESIGN.md).

libvirt and QEMU are known for broad operating-system compatibility: any Linux distribution, Windows, BSD and other CPU architectures. The goal is to bring that to an Android device as part of making it a full computer. It is a compatibility feature, not a performance one.

## Status

Design only. Nothing here runs yet, and there are open questions for the owner listed at the end of the design document.

## The plan

Two halves in one repository:

| | |
|---|---|
| **Root module** (`module/`) | A standard root-manager module, in the zip layout that Magisk, KernelSU and APatch all install. It owns libvirt: it ships `libvirtd` or `virtqemud` and `qemu-system-*`, starts them at boot, and exposes libvirt the usual way, over its UNIX sockets and optionally TCP or TLS. `virsh`, `virt-manager` over the network and any other libvirt client then work unchanged. |
| **droidtop plugin** (`droidtop-plugin/`) | Hooks the module's libvirt directly. Lists and manages virtual machines, shows them in [droidtop](https://github.com/Droidtop/droidtop) (Desktop mode, library), and offers VM management to other plugins. It requires the module and never reimplements it. |

## Acceleration

Where the device exposes `/dev/kvm` (pKVM or KVM-capable kernels), QEMU uses it. Everywhere else it falls back to QEMU's software emulation (TCG), which is slow but still runs.

## Licence

The code in this repository is GPL-3.0, see [LICENSE](LICENSE). Bundled libvirt (LGPL-2.1-or-later) and QEMU (GPL-2.0) keep their own licences, with their sources published alongside every release.
